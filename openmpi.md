How to use distributed memory via Openmpi
========================================


Scripts to obtain connected components
-------------------------------------
are described in [graph.md](https://bitbucket.org/eveng/tasks/src/master/graph.md)

If the size of the graph is very large, even the 400Gb of ram may
not suffice to analyze it. There various approaches to distribute
data over multiple nodes but OpenMPI is the most commonly used.

Fortunately, some of the Boost functions work with OpenMPI, and, in
particular, the connected_components function that we need.


Programming for OpenMPI is a bit (or, perhaps, a lot) of a hassle.
The same code is executed on each processor, but it may have its own
data, yet it is not clear when that is the case. The data needs to
be balanced among the nodes, that is another challenge.


[connectMpi.cpp](https://bitbucket.org/eveng/tasks/src/master/connectMpi.cpp)
is an implementation that appears to do the job.


Compilation
-----------

To compile an OpenMPI program a docker container with OpenMPI libraries/includes is
 needed (e..g., audris/openmpi). A bunch of libraries are needed for
 it to compile:
 ```
 mpiCC -O3 -o connectMpi connect.cpp -lboost_mpi \
   -lboost_serialization -lboost_system -lboost_graph \
   -lboost_graph_parallel
 ```

Running 
-------
Once compiled, the program needs to run on the specified containers.
OpenMPI communicates via pasword-less ssh, so being a real user may
be helpful. 

First we create a network interface (mpi) for each container to use
```
sdocker network create -d overlay --subnet=10.0.2.0/24 mpi
```

We then start the containers that run sssd, so that we
can log into each container 
```
seq 0 3 | while read i
do sdocker run  -id -v /da3_data/delta:/data \
  -v /home/audris:/home/audris \
  -e constraint:node==da$i.eecs.utk.edu \
  --hostname="mpi$i" audris/openmpi /bin/init.sh audris
```

Now there is one container on each server. /bin/init.sh
simply runs sssd so that we can become a user we want to be:
```
#!/bin/bash
sed -i 's/^$/+ : '$i' : ALL/' /etc/security/access.conf
/usr/sbin/sssd -fd 2
/usr/sbin/sshd -e
exec /bin/bash
```

We can log into one of the containers:
```
sdocker exec -it mpi1 su - audris
```

Once in, we can cd /data (mapped to /da3_data/delta). 
Ready to run? Not so simple, running an openmpi program requires
some parameters:
```
gunzip -c f2pNew.versions1 | \
 mpirun -np 4 --mca btl self,tcp \
   --mca btl_tcp_if_include eth0 \
   --host mpi0.mpi,mpi2.mpi,mpi3.mpi \
   ./connectMpi 13784514 | gzip > f2pNew.clones
```


So what are these mysterious parameters?
 1. "-np 4":  run on four nodes
 1. "--mca btl self,tcp": use tcp to communicate among nodes
 1. "--mca btl_tcp_if_include eth0": use interface eth0 to
    communicate to other mpi nodes.
 1. "--host mpi0.mpi,mpi2.mpi,mpi3.mpi" list of nodes. Note that
    docker appends the network name "mpi" to the host name, so
    mpi0.mpi is a fully qualified host name that is resolved within
    any container using overlay network "mpi"
 1. 13784514 is the number of nodes: $(gunzip f2pNew.names | wc -l)

