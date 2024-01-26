Graph databases
===============

Certain tasks may require queries that traverse a network of
links. Graph databases are supposed to help.

The following is the command to run docker container on any host
that has docker engine

```
docker run -d \
    -p 7474:7474 -p 7687:7687 \
    -v $(pwd)/neo4j/data:/data \
    -v $(pwd)/neo4j/logs:/logs \
    -v $(pwd):/import \
    --name yourpreferedname \
    neo4j:latest
```
The data will be residing on $(pwd) which will appear as /import
within the container. If it complains, you might need to remove existing container via
```
docker rm -f yourpreferedname
```

To connect to a container:
```
docker exec -it yourpreferedname bash
```

Once connected we can prepare the network data for
import, in this case links among developers in WoC based on shared projects
```
(echo "nID:ID,title"; zcat /da4_data/play/forks/P2AFullS.nA2A.100.names|sed 's|[,\r"]| |g' | sed "s|'| |" | awk '{print i++","$0}') > /data/play/forks/n.csv
(echo ":START_ID,:END_ID"; zcat /da4_data/play/forks/P2AFullS.nA2A.100.versions | sed 's| |,|')  > /data/play/forks/e.csv
bin/neo4j-admin import --nodes=nID=/data/n.csv --relationships=IN=/data/e.csv
```
  
Once the data is prepared: create and import the database:
  
```
./bin/neo4j console
ctrl-C
neo4j-admin set-initial-password  neo4j
./bin/neo4j-admin import --nodes=nID=/data/n.csv  --relationships=IN=/data/e.csv 
./bin/neo4j console

```
After that you need to remove the neo4j container and restart exactly as described above.


# CYPHER examples

A fairly good intro to Cypher is [the official one](https://neo4j.com/developer/cypher-query-language/)


Now lets run some queries, first a minor contributor:
```
MATCH v=(n {title:"Audris Mockus <audris@mockus.org>"})-[r:IN]-() return v;
```
If we want to simply count:
```
match v=(n {title:"Audris Mockus <audris@mockus.org>"})-[r:IN]-(a) return count(a)
```

Lets do something more sophisticated, find not immediate but
indirect connections:
```
match v=(n {title:"Audris Mockus <audris@mockus.org>"})-[:IN]-()-[:IN]-(a) return count(distinct(a))
```

There is a shortcut for that
 to specify two steps away: (*1..2) or we can simply add * for an arbitrary number of parents away. 
```
match v=(n {title:"Audris Mockus <audris@mockus.org>"})-[:IN*1..2]-(a) return count(distinct(a))
```


Multiple relationships can also be cobined (example is from the tutorial) with variables added
for output or where clause, as well as conditions imposed, for example recency:
```
    relationship-types like -[:KNOWS|:LIKE]->
    a variable name -[rel:KNOWS]-> before the colon
    additional properties -[{since:2010}]->
    structural information for paths of variable length -[:KNOWS*..4]->
```
