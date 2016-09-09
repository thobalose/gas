# gas

Graph-Aided Search

```sh
$ docker-compose up -d
$ docker-compose logs
$ curl -XPUT "http://localhost:9200/neo4j-index/_settings?index.gas.neo4j.hostname=http://localhost:7474&index.gas.enable=true"
```
You should see:

```sh
{"acknowledged":true}
```
Head on to [localhost:7474](http://localhost:7474) and start the import on the Neo4j browser:
```
CREATE CONSTRAINT ON (n:Movie) ASSERT n.objectId IS UNIQUE;

CREATE CONSTRAINT ON (n:User) ASSERT n.objectId IS UNIQUE;

USING PERIODIC COMMIT 500
LOAD CSV FROM "file:///ml-100k/u.user" AS line FIELDTERMINATOR '|'
CREATE (:User {objectId: toInt(line[0]), age: toInt(line[1]), gender: line[2], occupation: line[3]});

USING PERIODIC COMMIT 500
LOAD CSV FROM "file:///ml-100k/u.item" AS line FIELDTERMINATOR '|'
CREATE (:Movie {objectId: toInt(line[0]), title: line[1], date: line[2], imdblink: line[4]});
```
Check the ElasticSearch indices status
```sh
$ curl -XGET http://localhost:9200/_cat/indices
#yellow open neo4j-index 5 1 2625 0 475.3kb 475.3kb

$ curl -XGET http://localhost:9200/neo4j-index/User/_search
$ curl -XGET http://localhost:9200/neo4j-index/Movie/_search
```
Import relationships:
```
USING PERIODIC COMMIT 500
LOAD CSV FROM "file:///ml-100k/u.data" AS line FIELDTERMINATOR '\t'
MATCH (u:User {objectId: toInt(line[1])})
MATCH (p:Movie {objectId: toInt(line[0])})
CREATE UNIQUE (u)-[:LIKES {rate: ROUND(toFloat(line[2])), timestamp: line[3]}]->(p);
```
