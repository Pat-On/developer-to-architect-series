# Cassandra Configuration

## How we created it
```Dockerfile
# Set the base image to mongo
FROM cassandra:4.1

# File Author / Maintainer
MAINTAINER Anurag Yadav

# Install utility tools
RUN apt-get update && \
    apt-get install -y net-tools && \
    apt-get install -y iputils-ping && \
    apt-get install -y dnsutils && \
    apt-get install -y curl

COPY ./image/docker-entrypoint-second.sh /docker-entrypoint-second.sh
COPY ./image/insert-data.cql /insert-data.cql
COPY ./image/create-schema.cql /create-schema.cql
COPY ./image/check-readiness.sh /check-readiness.sh
COPY ./image/jmx_prometheus_javaagent.jar /opt/cassandra/lib
COPY ./image/jmx-prometheus.yml /etc/cassandra

RUN echo 'JVM_OPTS="$JVM_OPTS -javaagent:/opt/cassandra/lib/jmx_prometheus_javaagent.jar=7070:/etc/cassandra/jmx-prometheus.yml"' >> /etc/cassandra/cassandra-env.sh

ENTRYPOINT ["/docker-entrypoint-second.sh"]
CMD ["cassandra", "-f"]
```


## bash

```bash
#!/bin/bash

if [ -z "${REPLICATION_FACTOR}" ]; then
    REPLICATION_FACTOR=1
fi
# only one container is going to be data seeder
if [ -z "${SCHEMA_SEED_INSTANCE}" ]; then
    SCHEMA_SEED_INSTANCE="cassandra"
fi

CMD="cqlsh -f ./create-schema.cql"
SLEEP_DURATION=5
function create_schema {
    $CMD
    EXIT_CODE=$?
    while [ $EXIT_CODE != 0 ]; do
        sleep $SLEEP_DURATION
        $CMD
        EXIT_CODE=$?
	if [ $EXIT_CODE == 0 ]; then
	    echo "++++++++++++++++++++ SCHEMA CREATED ++++++++++++++++++++"
	elif [ $EXIT_CODE == 2 ]; then
            echo "?????????? Schema script error or schema already exists. Aborting. ??????????"
            exit 2
        else
	    echo "-------------- Cqlsh client connect error. Exit code: $EXIT_CODE  ---------------"
	    echo "-------------- Will try again connecting after $SLEEP_DURATION sec.    ---------------"
	fi
    done
}

sed -i -e 's/#REPLICATION_FACTOR#/'${REPLICATION_FACTOR}'/g' /create-schema.cql

if [[ $(hostname -s) = ${SCHEMA_SEED_INSTANCE} ]]; then
    create_schema &
fi

exec /usr/local/bin/docker-entrypoint.sh "$@"


```


### env 
```
# Services
server.port=8080
# DB Type Options -> SQL, CQL
database.type=CQL
database.cassandra.hosts=cassandra-1,cassandra-2,cassandra-3
database.cassandra.port=9042
database.cassandra.keySpace=oms
WAIT_TIME_FOR_DB=60
```


### docker-compose
- volumes
  - logs

```yml
  cassandra-1:
    build:
      context: ../../cassandra
      dockerfile: Dockerfile
    image: ntw/cassandra
    container_name: cassandra-1
    hostname: cassandra-1
    networks:
      - mynet1
    ports:
      - "9042:9042"
      - "7070:7070"
    environment:
      - JVM_OPTS=-Xms512M -Xmx1024M
      - SCHEMA_SEED_INSTANCE=cassandra-1
      - REPLICATION_FACTOR=3
      - CASSANDRA_SEEDS=cassandra-1
    volumes:
      - cassandra-1-logs:/var/log/cassandra
      - cassandra-1-data:/var/lib/cassandra
    entrypoint: /docker-entrypoint-second.sh

```