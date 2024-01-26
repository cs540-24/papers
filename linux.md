Linux Kernel History
====================

Start by cloning the repository and extracting commit log:

```
git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git linux
git log --all --numstat -M -C --full-history
--pretty=tformat:"STARTOFTHECOMMIT:linux%n%H;%T;%P;%an;%ae;%at;%cn;%ce;%ct;%s%n%b%nNOTES%N"| \
  gzip > linux.log.gz
```


Obtain delta (commit/file) and data for MAINTAINERS file
```
gunzip -c linux.log.gz | perl extrgitPlog.perl | gzip > linux.delta.gz
gunzip -c linux.delta.gz | grep ';MAINTAINERS;' | sort -t\; -k10 -n > MAINTAINERS.delta
```

Create Year/month from timestamps:
```
library(zoo);
x = read.table("MAINTAINERS.delta",sep=";",comment.char="", quote="");
x$V10 <- format(as.yearmon(as.POSIXct(x$V10, origin='1970-1-1')), '%Y.%m');
x$V11 <- format(as.yearmon(as.POSIXct(x$V11, origin='1970-1-1')), '%Y.%m');
write.table(x,file="MAINTAINERS.delta.1",sep=";", quote=F);
```

Obrtain miantainer for each month/file: quite time consuming!
```
cut -d\; -f3,11,12 MAINTAINERS.delta.1  | awk -F\; '{ if ($2>m) {print h";"m};h=$1;m=$2}END{print h";"m}' > MAINTAINERS.delta.m
cat MAINTAINERS.delta.m | grep -v '^;$' | while IFS=\; read h m
do git checkout --force $h
   find . -type f | sed 's|^\./||' | grep -v '^\.git/' | gzip >  MAINTAINERS.$h.$m.files.gz
   FF=scripts/get_maintainer.pl
   [[ -f $FF ]] && gunzip -c MAINTAINERS.$h.$m.files.gz | while read f
      do echo "$f;$m;"$($FF -f $f|perl -ane 's/;/SEMICOLON/g;s/\n/;/g;print')
      done 
done | gzip > MAINTAINERS.mtr.gz 
```

Convert timestamps to monts for all delta
```
library(zoo);
x = read.table("linux.delta.gz",sep=";",comment.char="", quote="",
col.names=c("prj", "v","tree","parent","an","cn","ae","ce","nadd","at","ct",
 "f","msg"),colClasses=c(rep("character",12)));
x$at[x$at < min(x$ct)] <- x$ct[x$at < min(x$ct)];
x$at <- format(as.yearmon(as.POSIXct(as.integer(x$at), origin='1970-1-1')), '%Y.%m');
x$ct <- format(as.yearmon(as.POSIXct(as.integer(x$ct), origin='1970-1-1')), '%Y.%m');
write.table(x[,c("f","v","an","ae","at","ct","nadd","msg")],gzfile("linux.delta.1.gz","w"),sep=";", quote=F, row.names=F,col.names=F);
```

Create a simple table: file;year;maintainer
```
use strict;
use warnings;
open A, "gunzip -c MAINTAINERS.mtr.gz|";
open B, ">mtr1";
while(<A>){
  chop();
  my ($f, $y, @m)=split(/\;/, $_, -1); 
  my @mtr=(); 
  for my $mt (@m){
    my $out = 0;
    if ($mt ne ""){
      $out++ if $mt =~ /(maintainer|supporter):/;
      $mt=~s/\([^\(\)]*\)\)$/\)/;
      $mt=~s/\([^\(\)]*\)$//; 
      $mt=~s/\> \(.*/>/;  
      $mt=~s/\s*$//; 
      push @mtr, $mt if $mt =~ m/[<>]/ && ($out || $y < 2011.01); #the maintainer:supporter start date
    }
  } 
  for my $m (@mtr){
    print B "$f;$y;".(scalar @mtr).";$m\n";
  }
  print B "$f;$y;0;torvalds\n" if ($#mtr < 0);
}
```

Merge maintainer info with delta
```
use strict;
use warnings;
my (%mtr, %mod, %m2mod);
open A, "mtr1";
while(<A>){
  chop();
  my ($f, $y, $nm, $m) = split(/\;/, $_, -1);
  my @fs = split (/\//, $f, -1);
  my $y1 = $y; $y1 =~ s/\.[0-9][0-9]$//;
  $mtr{$f}{$y}{$m}++;
  $mod{$y1}{$m}{$fs[0]}++;
}
close (A);

for my $y1 (keys %mod){
  for my $m (keys %{$mod{$y1}}){
    my @md = sort { $mod{$y1}{$m}{$b} <=> $mod{$y1}{$m}{$a} } (keys %{$mod{$y1}{$m}});
    $m2mod{$y1}{$m} = $md[0];
  }
}

open B, ">mtr2";
open A, "mtr1";
while(<A>){
  chop();
  my ($f, $y, $nm, $m) = split(/\;/, $_, -1);
  my $y1 = $y; $y1 =~ s/\.[0-9][0-9]$//;
  my @fs = split (/\//, $f, -1);
  print B "$f\;$fs[0]\;$m2mod{$y1}{$m}\;$y\;$nm\;$m\n";
}        
close (B);
     
open A, "delta";
open B, ">delta1";
while(<A>){
  chop();
  my ($f, $v, $a, $ae, $ya, $yc, @rest) = split(/\;/, $_, -1);
  my @ms = keys %{$mtr{$f}{$yc}};
  my $nm=$#ms+1;
  my $y1 = $yc; $y1 =~ s/\.[0-9][0-9]$//;
  for my $m (@ms){
    print B "$m\;$nm\;$f\;$v\;$m2mod{$y1}{$m};$a;$ae;$ya;$yc;".(join ';', @rest)."\n";
  } 
  my @fs = split (/\//, $f, -1);
  print B "torvalds\;0\;$f\;$v\;$fs[0];$a;$ae;$ya;$yc;".(join ';', @rest)."\n" if ($nm==0)  
}
```