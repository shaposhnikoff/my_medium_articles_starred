
# Elasticsearch Cluster and Kibana Using Docker Compose



Kick start your [Elasticsearch](https://www.elastic.co/) experiment, using [Docker](https://docs.docker.com/compose/) for your development project.

I’ve created three [Node](https://nodejs.org/) static Elasticsearch 7.5.1 clusters, using Docker Compose. Docker Compose also includes the new open sourced [Kibana](https://www.elastic.co/products/kibana) 7.5.1, running behind [NGINX](https://www.nginx.com/).

### Release notes

* [Elasticsearch 7.5.1 Release Notes](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/release-notes-7.5.1.html)

This cluster is not recommended to use in a production environment, but it provides you with a starting point.

## GitHub Repo
[**maxyermayank/docker-compose-elasticsearch-kibana**
*docker-compose-elasticsearch-kibana - Docker Compose for Elasticsearch and Kibana*github.com](https://github.com/maxyermayank/docker-compose-elasticsearch-kibana)

Services included in the GitHub repository:

* Three Node Elasticsearch version 7.5.1.

* Kibana version 7.5.1.

* Audit Beat version 7.5.1.

* Metric Beat version 7.5.1.

* Heart Beat version 7.5.1.

* Packet Beat version 7.5.1.

* File Beat version 7.5.1.

* APM Server version 7.5.1.

* APP Search version 7.5.1.

* NGINX.

## Requirements

* Docker 18.05

* Docker-compose 1.21

## Start Stack in Daemon Mode

    docker-compose up -d

## Check Status of Docker Compose Cluster

    docker-compose ps -a

## Cluster Node Info

    curl [http://localhost:9200/_nodes?pretty=true](http://localhost:9200/_nodes?pretty=true)

## Access Kibana

    [http://localhost:5601](http://localhost:5601)

## Accessing Kibana through NGINX

    [http://localhost:8080](http://localhost:8080)

## Access Elasticsearch

    [http://localhost:9200](http://localhost:9200)

## Access App Search

    [http://localhost:3](http://localhost:9200)002

## Resources
[**Hands on Elasticsearch**
*About Me: Application Architect at Oildex, a Services of Transzap Inc,.*medium.com](https://medium.com/@maxy_ermayank/hands-on-elasticsearch-8fa59d8aebfc)
[**Elasticsearch Resources**
*About Me: Application Architect at Oildex, a Services of Transzap Inc,*medium.com](https://medium.com/@maxy_ermayank/elasticsearch-resources-27d24f01c1dc)

I hope this post has helped you. Let me know what you think in the comments. Thanks for reading!
