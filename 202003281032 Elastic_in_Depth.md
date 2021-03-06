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

- **node.master** : synchronize
- **node.data** : store
- **node.ingest** : add doc to index, can do some transformation // simplify logstash pipeline
- **node.ml** : machine learning jobs
- **xpack.ml.enabled** : dedicated ML node
- **coordination** : (=all other conf to false) : coordinate queries, usefull for large queries, load balancer
- **node.voting_only** : participate to the vote for the master node. only usefull for large cluster

dim : data, ingest, master // default roles

=> done by optimizing the cluster to scale.


## query docs

- Create an index with settings:
```json
PUT /products 
{
	"settings":{
	"number_of_shards" : 2,
	"number_of_replicas": 2
	}
}
```

- Insert doc
`POST /products/_docs`

- Insert doc with ID
`PUT /products/_doc/<id>`

- Get doc
`GET /products/_doc/<id>`

- Update Value or add new field 
```json
POST /products/_update/<id>
{ "doc":{"key":"value"} }
```
-> it creates a new version of the doc, replaces the old one

- scripted update :
// update without knowing their value (decrease int for exemple)
```json
POST /products/_update/<id>
{
	"script":{
	"source": "ctx._source.stocks--"
	
	 OR 
	//"source": "ctx._source.stocks = 10"
	
	OR
	// "source": "ctx._source.stocks -= params.quantity"
	// "params":{ "quantity" : 4 }

	}
}
```
script can also handle value < 0 and not update (ctx.op = noop)

- Upsert
```json
POST /products/_update/<id>
{
	"script":{}, // if already exist, run script
	"upsert":{}  // if not exist, insert
}
```

- Replace
PUT /products/_update/<id>
{}

- Delete
DELETE /products/_update/<id>


## Routing

document goes in which shard ?
where is store the doc (shard)
Default routing strategy insure that the doc are evenly distributed in the shards

_routing will be added in the metada if you don't use the default routing. 
the default one uses the _id.
Adding shard to an index is not possible, we need to create a new index to do that.
The default routing use the nb of shards to choose, so increasing it will creatre issue retriving existing docs.

### how Elastic read doc
Elastic use routing to identify in which shard is the doc. 
Then use ARS (kind of load balancer) to identify the best relica in the group (in term of performance)

### How Elastic write doc
Elastic use routing to identify where to store the doc.
primary shard valid the mapping, store it locally and send the doc to the replicas 

primary terms : distinguish old and new primary shard by counting how many time the primary shard change
 each operation has the primary terms so elastic knows which operation need a refresh
 Elastic use checkpoint to speed up the process.

### Versionning

Elastic keep the number of revision but not the old version.
-> tell how many time the doc was modified

optimist Concurrency writting:
old way : use _version in the query to make fails query if already runs
new way: using primary term and sequence_number in the query
(POST /products/_update/<id>?if_primary_erm=X&if_seq_no=X)


### Update based on condition
```json
POST /products/update_by_query
{
	//"conflicts":"proceed",
	"script": {...},
	"query":{...}
}
```

- Create snapshot of the index
- search query on all search
- bulk request
- retry on error for each shard
- if > 10 errors, query aborded and not rollback

### Delete based on condition
```json
POST /products/delete_by_query
{
	"query":{...}
}
```

### Batch Processing

- *application/x-ndjson*
- end the line with \n
```json
POST /_bulk
{"index" :{ "_index" : "products", "_id" : 200 } } // will replace if already exist
{"name" : "<name>", "price" : "<p>", "key":"value"}
{ "create":{ "_index" : "products", "_id" : 201 } // will fail if already exist
{"name" : "<name>", "price" : "<p>", "key":"value"}
---
{"update" : { "_index" : "products", "_id" : 201 }}
{"doc" : "key":"update_value"}
{"delete" : { "_index" : "products", "_id" : 200 }}
```
--- 
or we can add the index in the URL and remove the "index" in the body



### Curl
```bash
Curl -H "Content-Type": "application/x-ndjson" -XPOST "http://localhost:9200/products/_bulk" --data-binary "@file.json"
```


## Mappings
- default number is long
- text != keyword, by default Elastic adds the two field 

### Meta field:
- _index : name of the index
- _id : Id of the doc
- _source: origin json oject
- _field_names: name of the field
- _routing : custom routing
- _version : number of revision of the doc
- _meta : store data when you can store w/e data not directly attached to the source

### Data type:

- Core: 

	* Text  (full text used to analyzed)
	* Keyword (filter & aggregation)
	* Numeric
	* Date 
	* Boolean
	* Binary 
	* Range
	
- Complex: 
	
	* Object : flatten object
	* Array : flatten array, need to have the same data type
	* Array of object : link between properties lost
	* Nested : independant document

- Geo:

	* geo_point : 
		location : lat;long || lat;long|| [lat;long]
	* geo_shape : point, polygon, linestring, multipoint, circle... 
		-> GeoJson
		
- Specialized:
	
	* IP
	* Completion (search-as-you-type)
	* Attachment (document, word, pdf)

### Add a mapping
```json
PUT /product/default/_mapping
{
	"properties":{"discount":{"type":"double"}}
}
```

### Change existing mapping
delete index 
create new one with new mapping
 
 
### Mapping parameter

- Coerce :  correct value ("5" -> 5) (true by default)
- copy_to: create a copy of the value to another properties (firstname + lastname -> fullname)
- dynamic 
- properties : wrapper of the field object. Same for nested object
- norms :  disable storage of norms / relevance
- format : specify format of date field
- null_value: replace null by value provided. 
- fields : index a field
	
if the mapping is not define correctly (and dynamic mapping = false)
your query can return nothing even if tey are correct because the value you are looking for are "hidden", no mapping available
update_by_query will update docs with a new mapping


## Text Analyze

each text from a doc a runs through an analyzer which tokenize the different terms and store it to an Inverted index
it's possible to change the strandard tokenizer. the default one use space, coma etc... to cut words et store it to an array.
then there are some function lowercase, stop words etc...

```json
POST /_analyze
{
	"tokenizer":"standard",
	"text": "I'm in the mood for drinking semi-dry red wine!"
}
```

### Inverted Index

Store text in a structure that makes the text easily queryable
mapping between terms & doc if they appear in it
where running a text query, elastic will use the inverted index instead of the doc


### Character Filters

- html_strip 
- mapping : replace some string
- pattern_replace : same but based on regular expression

### Tokenizers

- Word oriented

	* Standard
	* Letter Tokenizer (more primitive)
	* lowercase : same as letter but + lowercase
	* whitespace
	* UAX URL Email: preserve url & email 
	
- Partial words

	* N-Gram : go through letter by letter and increase
		 red wine -> re,red,ed,win,wine,in,ine,ne
	* Edge N-grame : same but only start of the beginning of a term	
		red wine -> re,red,wi,win,wine
		
- Structured Text
	* Keyword
	* Pattern : regular expression
	* Path : for filesystem path 
		/path, /path/to, /path/to/file


`PUT /existing_analyzer_config`


to mofdify an analyzer on an index, it will need to close it first to do the modification:
-> no read & write 

`POST /index/_close` -> index not available

// do the change of analyzer
// then
`POST /index/_open`  

# Search Query

## request URL 

* match everything: 

`GET /product/default/_search?q=*`

* match with looking for name with lobster :

`GET /product/default/_search?q=name:loster`

* With boolean: 

`GET /product/default/_search?q=tag:Meat AND name:Tuna`
*// whitespace ->%20*


## DSL query

* Leaf query vs Compound query (combinaison leaf query)

* basic query
```json
GET /product/default/_search
{
	"query": {
		"match_all" : { }
	}
}
```

### routing query
The node receiving the query is called coordating node, by default, everynode can be a coordinator
it broadcast the request on the other nodes. then get the result, merge them and send back the result

by default, the result are sort by score properties 
relevance determines how well the docs match the query (_score)

- **Term Frequency**: how many time the term appear in the doc

- **Inverse Document Frequency** : how many time the term appear in the index

- **Field Length Norm** : field length, the shorter the field, the more significanc it is
	
now, stop words are not removed, they are still used in the calculus -> nonlinear term frequency saturation
	
`"explain":true` -> have the explanation of the score, you can know hamy many documnent have certain terms etc...	

### debug a query
```json
GET /product/default/<id>/_explain
{
	"query": {	}
}
```

### Context

- **query**: How well document match

- **filter**: do document match (do not calculate relevance)
			filter on date, status, ...
			better perf :  cached filter

### term vs full text queries

- **term** : search for exact values -> look in the inverted index
			! careful to lowercase because index was tokenize
- **match** : the full text is analyze 

## Term level queries

*find exact matches*

```json
GET /product/default/_search
{
	"query": {
		"term": { "is_active": true }
		}
}
```

- multiple terms (will match any of the value provided)

```json
GET /product/default/_search
{
	"query": {
		"terms": { "tags.keyword": ["v1", "v2"]}
		}
}
```

- get many docs with ID

```json
GET /product/default/_search
{
	"query": {
		"ids": { "value": ["1", "2"]}
	}
}
```

- Get Doc with range
```json
GET /product/default/_search
{
	"query": {
		"range": {   
			"in_stock":{
				"gte" : 1,
				"lte" :5 
			} 
		}
	}
}
```
*! Works with date / can also specify the format*

### Date
```json
GET /product/default/_search
{
	"query": {
		"range": {   
			"created":{
				"gte" : "01-01-2010",
				"lte" : "31-12-2010",
				"format" : "dd-MM-yyy"
			} 
		}
	}
}
```
 - Date Math 
```json
GET /product/default/_search
{
	"query": {
		"range": {   
			"created":{
				"gte" : "01-01-2010+1d" // (add 1 day)
				//"gte" : "01-01-2010-2y" (remove 2 year)
				//"gte" : "01-01-2010/d" (rounded by the day)
				//"gte" : "01-01-2010/M +1d" (rounded by the month + add one day )
				//"gte" : "now/M +1d" 

			} 
		}
	}
}
```
[docs to date format](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#date-math)

### Non null value
any value that doesn't contains null -> exist
```json
GET /product/default/_search
{
	"query": {
		"exists": {   
			"field":"tags" 
			} 
		}
	}
}
``` 

### prefix
```json
GET /product/default/_search
{
	"query": {
		"prefix": {   
			"tags.keyword":"<prefix>" 
			} 
		}
	}
}
``` 

### Wildcare
```json
GET /product/default/_search
{
	"query": {
		"wildcard": {   
			"tags.keyword":"Veg*ble", // -> any caracters or text within
			"tags.keyword":"Veg?ble"  //-> match only 1 caracter
			} 
		}
	}
}
``` 
*! Query can be slow, avoid wildcard in the beginning*
*Careful, It can be ok in dev but be slow in prod with millions of docs*


### Regex
```json
GET /product/default/_search
{
	"query": {
		"regexp": { "tags.keyword": "<Regexp Here>" } 
		}
	}
}
```
*Based on Apache Lucene, not all feature of regexp are available*

## Full Text Query


### Flexible query

* `match` use OR by default between words

* To change the operator 
```json
GET /products/_search
{
  "query":{
    "match":{
      "title":{
        "query" : "Recipes with pasta or spaghetti",
        "operator": "and" // -> will look for the all the words "Recipes with pasta or spaghetti"
      }
    }
  }
}
``` 
* Order do not matter with match /^\\

* but will with match_phrase  \v/

```json
GET /products/_search
{
  "query":{
    "match_phrase":{
      "title":{
        "query" : "Recipes with pasta or spaghetti",
        "operator": "and" // -> will look for the all the words "Recipes with pasta or spaghetti"
      }
    }
  }
}
``` 

* Looking for many field at once
```json
GET /products/_search
{
  "query":{
    "multi_match":{
        "query" : "pasta",
        "fields": ["title", "description"] // will look for pasta in the title and the description
      }
    }
  }
}
``` 


## Compound Query

Bool query is kinda similar to the where claue in SQL

```json
GET /products/_search
{
  "query":{
    "bool":{
		"must" :[   // Keep score
			{Query1}
		],
		"must_not":[ // Keep score
			{Query}
		],
		"should":[ // Boost score if match but not required // it's like preferences in the result
			{Query}
		],
		"filtrer" :[ // Filter, do not care about the score
			{query}
		]
      }
    }
  }
}
```

**Should** : 

- if in a query context -> it will boost score
- Filtrer context : Is considered as a OR
- Filter without a must and one query in sould -> become a required element

### Debug 

We can name our object query:
```json
"must":{
	"match":{
		"ingredient.name": {
			"query" : "parmesan",
			"_name" : "parmesan_must"
		}
	}
}
```
*In the result, there will be an object called "matched queries" which give you the name of every object that match your query.
It's usefull to debug the should part.*

### Match Decomposed
```json
"query" :{
	"match" : { "title" : "pasta carbonara"}
}
 
== Same as == 

"query":{
	"bool":{
		"should":[
			{"term" : {"title" : "pasta"}},
			{"term" : {"title" : "carbonara"}},
		]
	}
}
```

AND similarly

```json
"query" :{
	"match" : {
	  "title" : { 
		  "query" :"pasta carbonara",
	   	  "operator" : "AND"}
	   }
}
 
== Same as == 

"query":{
	"bool":{
		"must":[
			{"term" : {"title" : "pasta"}},
			{"term" : {"title" : "carbonara"}},
		]
	}
}
```


## Joining query

Join between docs are expensive (in performance)
We basically sacrifice some disk space to increase search performance.

### Nested

In the mapping, a type is "nested" means that's another object within the bject itself:
```json
PUT /department/
{
	"mappings":{ "_doc": { "properties" :{
		"employees" : {"type": "nested"},
		"name" : {"type" : "text"}
	}
	}}
}

---
POST /department/_doc/1
{
	"name": "Development",
	"employees":[
		{"name":"Jon","age":39,"gender":"M"},
		{...}
	]
}
```
-> Keep association between docs.
Nested query should have two parameter:
- a path to the object 
- a query to run against this object

```json
GET /department/
{
	"query":{
		"nested":{
			"path": "employees",
			"inner_hits":{}, // include result for the upper level (here department)
			"query":{
				"bool":{
					"must":[
					{"match":{"employees.gender.keyword" : "M"}}
					]
				}
			}
		}
	}
}
```

### Multi Level Relation 

*1 company have 0-N department ; 0-N supplier and department have 0-N employees*
```json
PUT /company
{
	"mappings":{"_doc":{"properties":{
		"<join_field_name>":{
			"type" : "join",
			"relations":{
				"company" : ["department", "supplier"],
				"department" : "employee"
			}
		}
	}
}}
}
```

*insert a grand-child with a multi level relationship*
*Company ID : 1 ; Department ID : 2 ; Employee ID 3*
**We need to store it on the same shard has the grand parent hence the routing=1**
```json
PUT /company/_doc/3?routing=1
{
	"name": "Jon Doe",
	"<join_field_name>":{
		"name":"employee",
		"parent" : 2
	}
}
```

*Find any company that match the query (find any employee that is named Jon Doe)*
```json
GET /company/_search
{
	"query":{
		"has_child":{
			"type":"department",
			"inner_hits":{}, //If I want the intermediary result
			"query":{
				"has_child":{
					"type":"employee",
					"query":{
						"term":{
							"name.keyword" :"Jon Doe"
						}
					}
				}
			}
		}
	}
}
```

### Join Field
To be define in the mapping

```json
PUT /department/
{
	"mappings":{ "_doc": { "properties" :{
		"<join_field_name>" :{
			"type": "join",
			"relation":{
				"department" : "employee" // department is the parent of employee
			}
	}
	}}}
}
```


*defining the parent*
```json
PUT /department/_doc/1
{
	"name": "Development",
	"<join_field_name>" : "department" // precise the side of the relationship
}
```

**parent and Child must be stored on the same shard. 
we give the routing value to make sure this happens**

*Defining the Child*
```json
POST /department/_doc/2?routing=1
{
	"name": "Jon Doe",
	"age":39,
	"gender":"M",
	"<join_field_name>" :{ // precise the side of the relationship
			"name" : "employee", 
			"parent": 1
	}
}
```

*Query All employee from 1 department using the ID*
```json
GET /department/_search
{
	"query":{
		"parent_id":{
			"type": "employee",
			"id":1
		}
	}
}
```

*Query All employee from by searching a classic query (can be many departments for exemple*
```json
GET /department/_search
{
	"query":{
		"has_parent":{
			"parent_type": "department",
			"score":true, // use the releveant score for the child in the parent
			 "query":{
				 "term":{"name.keyword":"Development"}
			 }
		}
	}
}
```

*The oposite : matching parent when specific criteria from the child //any query*
*Ex: looking for department having employee over 50 and give boost score for male employee*
```json
GET /department/_search
{
	"query":{
		"has_child":{
			"parent_type": "employee",
			"score_mode" : "sum",
			"min_children": 1,
			"max_children" : 5
			 "query":{
				"bool":{
					"must":[
						{"range":{"age":{"gte": 50}}}
					],
					"should":[
						{"term":{"gender.keyword" :"M"}}
					]
				}
			 }
		}
	}
}
```
* score mode :
	- min
	- max
	- avg
	- sum
	- none (default)
* min & max children filter the result by putting a range on the count


### Lookup query

we have 2 objects :
-  stories 
	* user
	* content
-  users: 
	* name
	* following[user]

*We are looking for the stories of the users the user 1 is following*
```json
GET /stories/_search
{
	"query":{
		"terms":{
			"user":{
				"index":"users",
				"type":"_doc",
				"id" : 1,
				"path":"following"
			}
		}
	}
}
```
> Side note: this way of doing is better in terms of performance then doing 2 query in an application because it uses less bandwith and data transfer

### A word on performance join

* expensive in term of perf
* the more child / parent , the slower
* Each level of relation add complexity and time


##  result

### Format

* Get YML result

`GET /my_index/default/_search?format=yml
`
* Get a pretty result

`GET /my_index/default/_search?pretty
`

> use way more resources.

### Filter

- Do not get the sources, only get ID and metadata
```json
GET /my_index/default/_search
{
	"_source":false,
	"query":{}
}
```

- Only get the field in the result, works with nested object : field.child, works with wildcard
```json
GET /my_index/default/_search
{
	"_source":"<field>",
	"query":{}
}
```
- Customize what you want to get back in your result
```json
GET /my_index/default/_search
{
	"_source":{
		"include":"<name>",
		"exclude":"<name2>"
	},
	"query":{}
}
```

- Limit hits
```json
GET /my_index/default/_search
{
	"size":2,
	"query":{...}
}
```
> 10 by default

- Offeset, pagination
```json
GET /my_index/default/_search
{
	"size":2,
	"from":2, 
	"query":{...}
}
```

!= from a cursor.
If a modification happens (creation, modification), each query/ page will take it into account.


### Sorting

```json
GET /my_index/default/_search
{
	"query":{"match_all":{}},
	"sort":[
		{"<field>":"desc" },
		{"<field2>.<nestedPropertie>":"asc" },
		{"<field3>":{
			"type":"asc",
			"mode":	"avg"}
		}
	]
}
```

## Aggregation

### Metrics Aggregration

```json
GET /myindex/_search
{
	"size":0,
	"aggs":{
		"<agr_name>":{
			"<agr_type>":{
				"<agr_parameter>" // ex : "field" : "amount"
			}
		},
		"<agr_name2>":{
			"<agr_type>":{
				"<agr_parameter>" // ex : "field" : "amount"
			}
		}
	}
}
```

- **single value** 

Aggregation type

	* Sum 
	* Avg
	* Min
	* Max
	* Cardinality // *unique Count of a field ~ approximate number*
	* value_count //  *number of value the aggregation use* 
	* stats // all in one -> count + min + max + avg + sum


- **bucket aggregation**

	* term:

```json
GET /my_index/default/_search
{
	"size":0,
	"aggs" :{
		"status_terms":{
			"terms":{
				"field" : "<term.keyword>",
				"missing" : "<name>", // other value,
				"min_doc_count" : 0, // show only bucket with a minimum of doc
				"order":{			// order the buckets
					"_key" : "asc"
				}

			}
		}
	}
}
```

For data stored in multiple shard :

The query using number in bucket can be a bit misleading if the size is low. the query will take the top x of each shard to do it's aggregation. Tis top can be different from one shart to another, so the numnber may not be very accurate at some point. Improving size will slower the query but the result will be better

* **Nested aggregation**

```json
GET /my_index/default/_search
{
	"query":{},
	"size":0,
	"aggs" :{
		"status_terms":{
			"terms":{
				"field" : "<term.keyword>",
				"aggs":{
					"<bucket_name>":{
						"<bucket_type>":{
							"field" : "<field.keyword>"
						}
					}
				}
			}
		}
	}
}
```

* **filter aggregation**

- Simple one
```json
GET /my_index/default/_search
{
	"query":{},
	"size":0,
	"aggs" :{
		"<filter_name>":{
			"range":{ ...}
		},
		"aggs":{...} // this aggregation will run under the context of the filter
	}
}
```

- With rules

```json
GET /my_index/default/_search
{
	"size":0,
	"aggs" :{
		"<filter_name>":{
			"filters":{ 
				"filters":{ 
					"<field>":{
						"match" : {...}
					},
					"<field2>":{
						"match" : {...}
					}
				}
			},
			"aggs":{
				"<aggr_name>":{
					"<aggr_type>":{...}
			}
		}
	}
}
```
=> This will create
a main bucket filter_name with two bucket field & field2, both of them will have aggr_name inside


## Other

### Synonyms

The search is working with the term, not the full text query
In the mapping, there is a synonyms category in te settings:
```json
PUT /<index>
{
	"settings" : {"analysis":{"filter":{
		"<synonym>":{
			"type":"synonym",
			"synonyms":[
				"awful" => "terrible",
				"awesome" => "great","super" 
				"elasticsearch","logstash","kibana" => "elk"
				],
			"synonyms_path" : "file to synonyms"
		}
	}}}
}
```
- *replace awful by terrible*

- *gret and super will have the same position in the inverted index*

- *Careful to the place of the filter, run lowercase before synonym for instance*

- *When searching, the match, will go though the same analysis so you can search for great, even if the doc don't have the word itself, the doc can be match because it has awesome instead*

- for the file: 
	* 1 rule per line
	* Should be available on all nodes storing the index
	* when updating the synonym file, the old document won't have the synonym, it doesn't reindex all docs ! -> update by query too  "replay" the docs

### Highliter
```json
GET /index/default/_search
{
	"query":{...},
	"highlight":{
		"pre_tags":["<strong>"],
		"post_tag":["</strong"],
		"fields":{}
		}
}
```
will return fragment of content, the term will be engloble with <em> by default (strong, in our exemple) so we can find where your term is. (to find easily find it or even find the where is the synonym)
