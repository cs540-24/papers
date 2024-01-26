Running cassandra in a docker swarm
=======

/data/cassandra is a local path on each server

Overlay network for cassandra
====

```
sdocker network create -d overlay --subnet=10.42.42.0/24 cassandra
```

- Now check it out
```
sdocker run -d --name=csdr3 -p 7000:7000 -e constraint:node==da3.eecs.utk.edu -e "CASSANDRA_BROADCAST_ADDRESS=csdr3" -v /data/cassandra:/var/lib/cassandra --net=cassandra cassandra:latest 

sdocker run --name=csdr1 -d -p 7000:7000 -e constraint:node==da1.eecs.utk.edu -e "CASSANDRA_BROADCAST_ADDRESS=csdr1" -e "CASSANDRA_SEEDS=csdr3" -v /data/cassandra:/var/lib/cassandra --net=cassandra cassandra:latest 

#start containers on different nodes
sdocker run --name csdr0 -v /data/cassandra:/var/lib/cassandra -e constraint:node==da0.eecs.utk.edu -e "CASSANDRA_SEEDS=csdr3" -v /data/cassandra:/var/lib/cassandra -d --net=cassandra cassandra:latest

sdocker run --name csdr2 -v /data/cassandra:/var/lib/cassandra -e constraint:node==da2.eecs.utk.edu -e "CASSANDRA_SEEDS=csdr3" -v /data/cassandra:/var/lib/cassandra -d --net=cassandra cassandra:latest


sdocker run -it --rm --net=cassandra cassandra sh -c 'exec cqlsh csdr3'

CREATE KEYSPACE cmtlog
WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 2 };

USE cmtlog;
CREATE TABLE m2id (
  msg text PRIMARY KEY,
  id int,
);
CREATE TABLE msg (
  mid int,
  len int,
  hash text,
  name text,
  email text,
  time int,
  prj text,
  source text,
  PRIMARY KEY (mid,hash,name,email,time,prj,source)
);
```

Are they connected?
```
sdocker exec -it shell2 /bin/sh
/ # ping shell1
PING shell1 (10.0.1.2): 56 data bytes
64 bytes from 10.0.1.2: seq=0 ttl=64 time=0.276 ms
```
Voila!


