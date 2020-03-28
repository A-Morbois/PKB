# Elastic_in_Depth
tags = #ELK #ElasticSearch


# Intro
Often used as :
 - an application performance Management (APM)
 - Event
 - forecast
 - anomality detection
 - ...

A document in Elastic = 1 row in relational db
1 field = 1 column

write in Java / Apache Lucene / highly scalable


* **Logstash** : ETL
* **Kibana** : visualization
* **XPACK**: 
	- authen / security
	- permission / authorization
	- ELK monitoring (cpu, memory usage, space,...)
	- Alerting
	- Reporting (export report, trigger, customize)
	- Machine Learning (anomality, forecast)
	- Graph (relationship of data / suggestion, related // relevance != popular)
	- SQL 
* **Beats**: agent that collect data (filebeat, metricbeat, packetbeat,winlogbeat,auditbeat, heartbeat)


Elastic store default configuration for Kibana

## Installation

for the deployment on the cloud, there are different templates with differente configuration based on what the needs are.
or you can choose a custom config.

CloudID : need to be sent each time to identify your node

## Architecture

- **Document** : piece of information : data (json)
- **index** : groupe doc logically / collection of documents
- **Node** : instance of Elasticsearch containing part of the data.
- **Cluster**: collection of nodes / they are completly independant (data / configuration).

* info about the cluster
`GET /_cluster/health `

* info readable
`GET /_cat/nodes?v`
`GET /_indices/nodes?v`
`GET /_nodes/`
`GET /_cat/shards?v`

### Shard
an index can be cuted in shard. Each shard is (kind of) independant
scale data storage
parallel queries -> better perf

under 7.0 -> 5 shard per index by default
above 7.0 -> 1 shard --

Split -> increase number of shard
Shrink -> decrease

~~ 5 shard if more > millions doc 

### Replication / snapshoot

enabled by default (if many node).
create copy of shard : named **replica**
replica shard are never stored on the same node of the primary shard.
replicate once for not really critical // more if loss is not acceptable
replicais for live data


snapshots can be for an index / cluster 
snapshot export data to a file. use to restore, rollback

increase replica = better perf on the index
a replica is a fully usable shard
yellow state for an index : replica not allocated
auto-expand replica : Elastic create replica automaticly if needed


### Adding a new node

specify a name of your node (node.name in elasticsearch.yml)
`GET /_cluster/health ` -> get your number of node / shards etc...

overwrite settings of your node when starting it (instead of copy & pasting the directory and change the config file) :
`bin/elasticsearch -Enode.name=node-3 -Epath.data=./node-3/data -Epath.logs`

### Node roles

**node.master** : synchronize
**node.data** : store
**node.ingest** : add doc to index, can do some transformation // simplify logstash pipeline
**node.ml** : machine learning jobs
**xpack.ml.enabled** : dedicated ML node
**coordination** : (=all other conf to false) : coordinate queries, usefull for large queries, load balancer
**node.voting_only** : participate to the vote for the master node. only usefull for large cluster

dim : data, ingest, master // default roles

=> done by optimizing the cluster to scale.