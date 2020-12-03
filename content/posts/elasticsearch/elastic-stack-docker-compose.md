---
title: ELK Stack using Docker Compose
date: 2020-12-03
tags:
- ELK
- Docker-Compose
categories:
- docker
keywords:
    - Elasticsearch
    - Docker compose
---

The Elastic Docker registry contains Docker images for all the products in the Elastic Stack: https://www.docker.elastic.co/.

## Run with docker compose

To get the default distributions of Elasticsearch and Kibana up and running in Docker, you can use Docker Compose.

1. Create a docker-compose.yml file for the Elastic Stack. \
   The following example brings up a single node cluster and Kibana so you can see how things work. 
   This all-in-one configuration is a handy way to bring up your first dev cluster.

```s
version: '3.7'

services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.10.0
    container_name: es01
    environment:
    - node.name=es01
    - cluster.name=es-docker-cluster
    - cluster.initial_master_nodes=es01
    - bootstrap.memory_lock=true
    - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
    healthcheck:
      test: 
      - "CMD-SHELL"
      - "curl -s https://localhost:9200 >/dev/null; if [[ $$? == 52 ]]; then echo 0; else echo 1; fi"
      interval: "30s"
      timeout: "10s"
      retries: 5
  
  kib01:
    image: docker.elastic.co/kibana/kibana:7.10.0
    container_name: kib01
    depends_on: ["es01"]
    ports:
      - 5601:5601
    environment:
      ELASTICSEARCH_URL: "http://es01:9200"
      ELASTICSEARCH_HOSTS: "http://es01:9200"
    networks:
      - elastic

volumes:
  data01:
    driver: local

networks:
  elastic:
    driver: bridge
```



## Run with TLS enabled using docker compose

[TLS](/posts/elasticsearch/elastic-tls-docker-compose)
