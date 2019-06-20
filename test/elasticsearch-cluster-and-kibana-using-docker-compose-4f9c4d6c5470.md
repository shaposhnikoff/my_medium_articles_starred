
# Elasticsearch Cluster and Kibana using docker-compose

About Me: Application Architect at Oildex, a Services of Transzap Inc,

Kick start your Elasticsearch experiment using docker for your development project. I have created 3 Node static Elasticsearch 7.0.0 cluster using docker-compose. Docker-compose also includes new Open Sourced Kibana 7.0.0 running behind Nginx.

![](https://cdn-images-1.medium.com/max/2000/1*uHeGO2Hv9q8fIniurDhSAQ.png)

[Elastic Stack 7.0.0 Release Notes](https://www.elastic.co/guide/en/cloud/current/ec-release-notes-2019-04-10.html)

[Elasticsearch 7.0.0 Release Notes](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/release-notes-7.0.0.html)
> # This Cluster is not recommended to use for production environment but provides you starting point.

## Github Repo
[**maxyermayank/docker-compose-elasticsearch-kibana**
*docker-compose-elasticsearch-kibana - Docker Compose for Elasticsearch and Kibana*github.com](https://github.com/maxyermayank/docker-compose-elasticsearch-kibana)

Services Included in GitHub Repository:

* 3 Node Elasticsearch version 7.0.0

* Kibana version 7.0.0

* Audit Beat version 7.0.0

* Metric Beat version 7.0.0

* Heart Beat version 7.0.0

* Packet Beat version 7.0.0

* File Beat version 7.0.0

* APM Server version 7.0.0

* NGINX

### Requirements

* Docker 18.05

* Docker-compose 1.21

### Start Stack in Daemon Mode

    docker-compose up -d

### Check status of docker-compose cluster

    docker-compose ps -a

### Cluster Node Info

    curl [http://localhost:9200/_nodes?pretty=true](http://localhost:9200/_nodes?pretty=true)

### Access Kibana

    [http://localhost:5601](http://localhost:5601)

### Accessing Kibana through Nginx

    [http://localhost:8080](http://localhost:8080)

### Access Elasticsearch

    [http://localhost:9200](http://localhost:9200)

## Resources
[**Hands on Elasticsearch**
*About Me: Application Architect at Oildex, a Services of Transzap Inc,.*medium.com](https://medium.com/@maxy_ermayank/hands-on-elasticsearch-8fa59d8aebfc)
[**Elasticsearch Resources**
*About Me: Application Architect at Oildex, a Services of Transzap Inc,*medium.com](https://medium.com/@maxy_ermayank/elasticsearch-resources-27d24f01c1dc)

Also, check out my post on Open Distro Elasticsearch here:
[**TL;DR AWS — Open Distro Elasticsearch**
*Hands-on walk through of Open Distro Elasticsearch and High level comparison with Elastic.co Elasticsearch*medium.com](https://medium.com/@maxy_ermayank/tl-dr-aws-open-distro-elasticsearch-fc642f0e592a)

I hope this post has helped you. **If you enjoyed this article, please don’t forget to clap👏 !** I would love to know what you think and would appreciate your thoughts on this topic. You can also follow me on [Medium](https://medium.com/@maxy_ermayank), [GitHub](https://github.com/maxyermayank) and [Twitter](https://twitter.com/maxy_ermayank) for more updates.
