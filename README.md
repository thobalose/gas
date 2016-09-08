# gas

Graph-Aided Search

```
$ docker-compose up
$ curl -XPUT "http://localhost:9200/neo4j-index/_settings?index.gas.neo4j.hostname=http://localhost:7474&index.gas.enable=true"
```

You should see:

```
{"acknowledged":true}
```
