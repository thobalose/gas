FROM neo4j:2.3.3
MAINTAINER Thoba Lose "lose.thoba@gmail.com"

ENV GRAPHAWARE_PROD_URL http://products.graphaware.com/download
ENV DATASETS http://files.grouplens.org/datasets/movielens/ml-100k.zip

RUN wget ${GRAPHAWARE_PROD_URL}/framework-server-community/graphaware-server-community-all-2.3.3.37.jar \
  -P plugins/ \
  && wget ${GRAPHAWARE_PROD_URL}/neo4j-to-elasticsearch/graphaware-neo4j-to-elasticsearch-2.3.3.37.1.jar \
  -P plugins/

COPY neo4j-server.properties conf/

RUN sed -i '/load_csv_file_url_root/s/^#//;s/data\/import/\/import/' conf/neo4j.properties

RUN mkdir /import \
  && wget ${DATASETS} -P /import/ \
  && unzip /import/ml-100k.zip -d /import/ \
  && rm /import/ml-100k.zip
