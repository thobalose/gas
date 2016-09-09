# gas

A `docker-compose` setup for the [Graph-Aided Search](http://graphaware.com/neo4j/2016/04/20/graph-aided-search-the-rise-of-personalised-content.html) demo.

This setup will automate the configuration and start-up of:
* ElasticSearch `v2.3.2` on port `9200` with the [Graph-Aided-Search plugin](https://github.com/graphaware/graph-aided-search)
* Neo4j `v2.3.3` database on port `7474` with the [Graphaware Framework](http://products.graphaware.com/?dir=framework-server-community) and [Elasticsearch Plugin](https://github.com/graphaware/neo4j-to-elasticsearch), and
* CSV sources in the `/import` directory of Neo4j

### Getting up and running

```sh
$ docker-compose up -d
$ docker-compose logs
```
Configure GAS by defining Neo4j URL and enabling it:

```sh
$ curl -XPUT "http://localhost:9200/neo4j-index/_settings?index.gas.neo4j.hostname=http://localhost:7474&index.gas.enable=true"
```
You should see:

```sh
{"acknowledged":true}
```
*If you get:*

```sh
{"error":{"root_cause":[{"type":"index_not_found_exception","reason":"no such index","resource.type":"index_or_alias","resource.id":"neo4j-index","index":"neo4j-index"}],"type":"index_not_found_exception","reason":"no such index","resource.type":"index_or_alias","resource.id":"neo4j-index","index":"neo4j-index"},"status":404}t
```
Create the index with curl and re-run:

```sh
$ curl -XPUT 'http://localhost:9200/neo4j-index'
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
Check the ElasticSearch indices status:

```sh
$ curl -XGET http://localhost:9200/_cat/indices
yellow open neo4j-index 5 1 2625 0 475.3kb 475.3kb

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

Try a simple query:

```sh
curl -X POST http://localhost:9200/neo4j-index/Movie/_search -d '{
  "query" : {
      "bool": {
        "should": [{"match": {"title": "love"}}]
      }
  }
}';
```
