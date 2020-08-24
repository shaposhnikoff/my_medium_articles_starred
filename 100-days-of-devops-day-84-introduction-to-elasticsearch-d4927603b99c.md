
# 100 Days of DevOps — Day 84-Introduction to ElasticSearch

Welcome to Day 84 of 100 Days of DevOps, Focus for today is Introduction to ElasticSearch

*ElasticSearch is an ultrafast distributed(fault tolerant)search and analytics engine powered by Apache Lucene Project. ElasticSearch is specifically designed to search an index of massive datasets in the order of Petabytes.*

***Installing ElasticSearch***

***Requirement***

    *java >7*

***Installation***

    *$ wget [https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.4.2.zip](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.4.2.zip)*

    *$ unzip elasticsearch-5.4.2.zip*

    *$ cd elasticsearch-5.4.2*

    *# To start ElasticSearch*

    *$ bin/elasticsearch*

*To test elasticsearch*

    *$ curl [http://localhost:9200](http://localhost:9200)*

    *{
    “name” : “ASEUJl9”,*

    *“cluster_name” : “elasticsearch”,*

    *“cluster_uuid” : “LSEdI1HARj-O_cSk0k8DLg”,*

    *“version” : {*

    *“number” : “5.4.2”,*

    *“build_hash” : “929b078”,*

    *“build_date” : “2017–06–15T02:29:28.122Z”,*

    *“build_snapshot” : false,*

    *“lucene_version” : “6.5.1”*

    *},*

    *“tagline” : “You Know, for Search”*

    *}*

*Now if we want to test via GUI interface we need **Kibana** for that*

    *$ wget [https://artifacts.elastic.co/downloads/kibana/kibana-5.4.2-darwin-x86_64.tar.gz](https://artifacts.elastic.co/downloads/kibana/kibana-5.4.2-darwin-x86_64.tar.gz)*

    *$ tar -xvf kibana-5.4.2-darwin-x86_64.tar.gz*

    *$ cd kibana-5.4.2-darwin-x86_64*

    *$ vim config/kibana.yml*

    *Now go inside kibana.yml and uncomment this entry*

    ***elasticsearch.url: "http://localhost:9200"***

    *# Start Kibana*

    *$ bin/kibana*

*# To access Kibana*

[*http://localhost:5601/](http://localhost:5601/)*

![*Kibana*](https://cdn-images-1.medium.com/max/5728/1*UoP7wFWBnc4lh6k1JQlg2A.png)**Kibana**

*Now let’s try to put some index**(using Dev Tools)**(don’t worry if you don’t understand what index is)*

![](https://cdn-images-1.medium.com/max/5740/1*tjaESiJXQ_DGpXyMeEbyuQ.png)

*Now try to get it*

![](https://cdn-images-1.medium.com/max/5756/1*hZgYR6aydcBkA5TJzYN-5w.png)

***ElasticSearch Stack***

![](https://cdn-images-1.medium.com/max/2154/1*vlM2M3lBDV_Yhxm4JhbJVQ.png)

    ***Kibana:** Data Visualization*

    ***ElasticSearch:** Store,Index,Search and Analyze data*

    ***Logstash,Beats:** Data Ingestion*

    ***X-Pack: **Additional Features*

***What is an Index?***

*The index is a logical namespace which points to one or more shards(container for data)in an ElasticSearch cluster and it serves as the place to store related data*

    ***# Adding Index***

    ***$ curl -XPUT [http://localhost:9200/testing](http://localhost:9200/testing)***

    *{"acknowledged":true,"shards_acknowledged":true}*

    ***# Getting all indices in a cluster***

    ***$ curl -XGET [http://localhost:9200/_cat/indices?v](http://localhost:9200/_cat/indices?v)***

    *health status index   uuid                   **pri** **rep** docs.count docs.deleted store.size pri.store.size*

    *yellow open   testing Zkhf_-5pR2yPvjvqgdLlyg   5   1          0            0       650b           650b*

    *yellow open   test    rfTZJwiSTSmufAzzHWjSkg   5   1          0            0       650b           650b*

    *yellow open   .kibana 8-5xFao6TjCqV3a_iNCFnw   1   1          1            0      3.1kb          3.1kb*

    ***# Get a specific index in a cluster***

    ***$ curl -XGET [http://localhost:9200/testing?pretty](http://localhost:9200/testing?pretty)***

    *{*

    *"testing" : {*

    *"aliases" : { },*

    *"mappings" : { },*

    *"settings" : {*

    *"index" : {*

    *"creation_date" : "1498323861381",*

    *"number_of_shards" : "5",*

    *"number_of_replicas" : "1",*

    *"uuid" : "Zkhf_-5pR2yPvjvqgdLlyg",*

    *"version" : {*

    *"created" : "5040299"*

    *},*

    *"provided_name" : "testing"*

    *}*

    *}*

    *}*

    *}*

***NOTE:***

* *When Index is created by default it’s assigned to **5 primary shards**(fixed)*

* *There is a **one** replica shard(can be changed any time)*

***ElasticSearch VS Relational Database Analogy***

![](https://cdn-images-1.medium.com/max/2000/1*0gUMUCd81Oxu-npn-NZcTw.png)

***Documents***

*As mentioned above Document can think of Row(an individual entry in ElasticSearch) and contained in Document is Field which we can think as a column.*

*Easiest way to understand this*

![](https://cdn-images-1.medium.com/max/5028/1*IjdlpjnLz3Y7g6N-Yw718w.png)

    *index --> my_index
    type --> my_doc
    document --> 1
    Fields -->*

    *"name":"testuser",
              "years":1,
              "date":"2017-06-24"*

*Now to get the data back*

![](https://cdn-images-1.medium.com/max/5032/1*G5OlfZb4r874Pn8dxg6Sxw.png)

*Now to get a mapping*

![](https://cdn-images-1.medium.com/max/4992/1*qq5QC37Gl2RhftXYGVUVyQ.png)

*If we want to get information about a particular index*

    *$ curl -XGET [http://localhost:9200/my_index?pretty](http://localhost:9200/my_index?pretty)
    {*

    *“my_index” : {*

    *“aliases” : { },*

    *“mappings” : {*

    *“my_doc” : {*

    *“properties” : {*

    *“date” : {*

    *“type” : “date”*

    *},*

    *“name” : {*

    *“type” : “text”,*

    *“fields” : {*

    *“keyword” : {*

    *“type” : “keyword”,*

    *“ignore_above” : 256*

    *}*

    *}*

    *},*

    *“years” : {*

    *“type” : “long”*

    *}*

    *}*

    *}*

    *},*

    *“settings” : {*

    *“index” : {*

    *“creation_date” : “1498325421498”,*

    ***“number_of_shards” : “5”,***

    ***“number_of_replicas” : “1”,***

    *“uuid” : “upg2ElNdTHGpcCPCOMb6AA”,*

    *“version” : {*

    *“created” : “5040299”*

    *},*

    *“provided_name” : “my_index”*

    *}*

    *}*

    *}*

    *}*

*Now if we want to change the number of primary shards(not possible as they are **immutable**)and number of replicas, we can do it easily with the help of Kibana Developer Console*

![](https://cdn-images-1.medium.com/max/4756/1*WDgAvPOil_NP0IrNtsi4wA.png)

*To verify it*

![](https://cdn-images-1.medium.com/max/4992/1*w2GEI4qPer6l3XqyxvHJrg.png)

*As mentioned above a number of shards is **immutable**(i.e) we can’t change that value but we can change the number of replicas*

*So let’s try to change the number of replicas*

![](https://cdn-images-1.medium.com/max/3780/1*w64O3JIMY3JGuiaiE3dTBw.png)

*But if we try to change the number of shards*

![](https://cdn-images-1.medium.com/max/5024/1*eooUfuWkcuu7DbimGi6Ulg.png)

*To verify it*

![](https://cdn-images-1.medium.com/max/4992/1*R7rd34rOJkHoO42VjQ0kZw.png)

***Mapping***

*Mapping describes the field properties of a document through the type. Mapping should be setup before the first document added as the system knows what type of data each field contains.*

    *name: string*

    *date: date*

***NOTE:** If you try to change the datatype after it’s been indexed(eg: from date to string) all that data will become unsearchable. The only solution to this problem is to re-index all the data.*

*A simple example of Mapping*

![](https://cdn-images-1.medium.com/max/5004/1*QCbYAB3-eObK2Ycq2L177Q.png)

***Deleting an Index***

*First, let’s verify all index*

    *$ curl -XGET [http://localhost:9200/_cat/indices?v](http://localhost:9200/_cat/indices?v)*

    *health status **index** uuid pri rep docs.count docs.deleted store.size pri.store.size*

    *yellow open **testing** Zkhf_-5pR2yPvjvqgdLlyg 5 1 0 0 650b 650b*

    *yellow open **my_index** upg2ElNdTHGpcCPCOMb6AA 5 1 1 0 4.4kb 4.4kb*

    *yellow open **my_new_test** qcwTmf7VSbiutHfZOJSlZw 1 2 0 0 130b 130b*

    *yellow open **test** rfTZJwiSTSmufAzzHWjSkg 5 1 0 0 795b 795b*

    *yellow open **.kibana** 8–5xFao6TjCqV3a_iNCFnw 1 1 1 0 3.1kb 3.1kb*

*Now to delete an index*

    *$ curl -XDELETE [http://localhost:9200/test](http://localhost:9200/test)*

    *{“acknowledged”:true}*

*Deleting an index effectively removes all documents and types associated with that index.*

*To delete multiple index(via developer console)*

![](https://cdn-images-1.medium.com/max/5004/1*IjsvJYWPi97xTdpuOIp7kw.png)

*To check the cluster health*

    *$ curl -XGET [http://localhost:9200/_cluster/health?pretty](http://localhost:9200/_cluster/health?pretty)*

    *{*

    *“cluster_name” : “elasticsearch”,*

    *“status” : **“yellow”**, # It shows yellow because it's single node cluster*

    *“timed_out” : false,*

    *“number_of_nodes” : 1,*

    *“number_of_data_nodes” : 1,*

    *“active_primary_shards” : 12,*

    *“active_shards” : 12,*

    *“relocating_shards” : 0,*

    *“initializing_shards” : 0,*

    *“unassigned_shards” : 13,*

    *“delayed_unassigned_shards” : 0,*

    *“number_of_pending_tasks” : 0,*

    *“number_of_in_flight_fetch” : 0,*

    *“task_max_waiting_in_queue_millis” : 0,*

    *“active_shards_percent_as_number” : 48.0*

    *}*

***Adding a document***

* *We can add a document without assign an id, ElasticSearch by default assign id for us(we need to use **POST** for this purpose)*

![](https://cdn-images-1.medium.com/max/5028/1*nLZ2qWq31_nUuYrsy7SJYw.png)

* ***Optionally we can assign id using PUT(But if the same id exist, ElasticSearch will throw 409 conflict error)***

![](https://cdn-images-1.medium.com/max/5036/1*Lio4f4vjqwEK3_Zcjg-s9A.png)

***Let’s add a field to our document mapping, we can’t change existing mapping but we can add new mapping after the index has been created***

![](https://cdn-images-1.medium.com/max/4996/1*0TXltmFzLMZBawgYV8_UIg.png)

***To delete a document***

![](https://cdn-images-1.medium.com/max/5024/1*FuX2PDeBIUkxboH3vHzQnw.png)

***To verify it***

![](https://cdn-images-1.medium.com/max/5020/1*4k9KGduP9SWwvL20aYPhFQ.png)

***Bulk API***

* *Allow us to index multiple documents at one time*

![](https://cdn-images-1.medium.com/max/5012/1*FDmVbvONX4vedre4YJyoFg.png)

*To verify it*

![](https://cdn-images-1.medium.com/max/5008/1*ZhyMxpEiI5hmI58Oq4uWug.png)

*To use a bulk API to add external document*

    *curl -XPOST [http://localhost:9200/books/_bul](http://localhost:9200/books/_bulk)k --data-binary @test.json*

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
[**100 Days of DevOps — Day 83-Introduction to Splunk**
*Welcome to Day 83 of 100 Days of DevOps, Focus for today is Introduction to Splunk*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-83-introduction-to-splunk-9c1caf04f253)
