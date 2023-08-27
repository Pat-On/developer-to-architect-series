# Configuring of ElasticSearch, Flutentd and Kibana

- env files nothing changed


docker-compose files
```yml
services:

  fluentd:
    build:
      context: ../../fluentd
      dockerfile: Dockerfile
    image: ntw/fluentd
    container_name: fluentd
    hostname: fluentd
    networks:
      - mynet1
    ports:
      - "24224:24224"
    volumes:
      - ${PWD}/fluent.conf:/fluentd/etc/fluent.conf
      - fluentd-logs:/fluentd/log
    depends_on:
      - "elasticsearch"
    restart: unless-stopped

  
  elasticsearch:
    image: elasticsearch:7.13.2
    container_name: elasticsearch
    hostname: elasticsearch
    environment:
      - "discovery.type=single-node"
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "xpack.security.enabled=false"
    expose:
      - "9200"
    ports:
      - "9200:9200"
    networks:
      - mynet1
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    restart: unless-stopped
    

  kibana:
    image: kibana:7.13.2
    container_name: kibana
    hostname: kibana
    ports:
      - "5601:5601"
    networks:
      - mynet1
    environment:
      - ELASTICSEARCH_HOSTS="http://elasticsearch:9200"
    restart: unless-stopped
    depends_on:
      - "elasticsearch"
      
```

worth checking configuration files used top create this docker-compose services

## configuring docker images to send logs:
```yml
  web:
    build:
      context: ../../web
      dockerfile: Dockerfile
    image: ntw/web
    container_name: web
    hostname: web
    networks:
      - mynet1
    ports:
      - "8000:8000"
    command: python3 manage.py runserver 0.0.0.0:8000
    env_file: .env
    volumes:
      - app-logs:/var/log/oms
    logging:
      driver: "fluentd"
      options:
        fluentd-address: "127.0.0.1:24224"          <---- local host / replace with dns 
        tag: web
    depends_on:
      - "fluentd" 
```