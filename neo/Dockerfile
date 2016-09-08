FROM neo4j:2.3.3
MAINTAINER Thoba Lose "lose.thoba@gmail.com"

ENV GRAPHAWARE_PROD_URL http://products.graphaware.com/download

RUN wget ${GRAPHAWARE_PROD_URL}/framework-server-community/graphaware-server-community-all-2.3.3.37.jar \
  -P /var/lib/neo4j/plugins/

RUN wget ${GRAPHAWARE_PROD_URL}/neo4j-to-elasticsearch/graphaware-neo4j-to-elasticsearch-2.3.3.37.1.jar \
  -P /var/lib/neo4j/plugins/

COPY neo4j-server.properties conf/

EXPOSE 7474 7473

CMD ["neo4j"]
