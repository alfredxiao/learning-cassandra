# Welcome to Learning with Astra #

## My notes on learning Cassandra ##

- ACID
  * Not valid when considering master slave, unless slave does NOT serve requests at all?

- Relational DB
  * Vertical scale is limiting
  * Normalised form of tables is BAD for queries, esp. complex queries  
  * Sharding
    * No more joins
    * No more aggregations
    * Querying secondary indexes requires hitting every shard

RDBMS Lessons Learned & Cassandra Design Principles
* Consistency is not practical
  * So we give it up
* Manual Sharding & Rebalancing is hard
  * So build it as part of the cluster
* ??
  * Simplify the architecture - no more master/slave
* Scaling up is expensive
  * We go for horizontal scaling
* Scatter/gather is bad
  * Goal is to always hit 1 machine
  * Denormalise everything

Cassandra Characteristics
* Distributed
  * No Master/Slave
  * No Leader election  
  * No Failover
* High Availability
* Linear Scalability
* Predictable Performance
* Multi-DC
  * RF is per keyspace per DC
  * DC can be physical or logical
* Commodity Hardware
* CAP Tradeoffs
  * When Network Partitioning happens, we choose Availability over Consistency
* Consistency Level
  * Per query basis

Hash Ring (Cluster)
* No Master/slave
  * All nodes are equal
* No config server, zookeeper
* Data is partitioned around the ring
  * Location of data on ring is determined by partition key
* Data is replicated to RF=N servers
* All nodes hold data and can answer queries (both reads & writes)

Once table is defined
* You cannot change primary key!

Query Clustering Columns
* Partition Key must be provided
* = or range queries (> <) can be used on clustering columns
* All equality comparisons must come before inequality columns

Storage
* DO NOT run Cassandra on SAN
  * https://www.datastax.com/blog/2017/01/impact-shared-storage-apache-cassandra

Snitch???
* Is the mechanism to let Cassandra know which node is on which DC and Rack.
* All nodes must use the same snitch
* Changing cluster topology requires restarting all nodes
  * Run sequential repair and cleanup on each node

Logging Level
* ./nodetool getlogginglevels
* ./nodetool setlogginglevel org.apache.cassandra TRACE
* /var/log/cassandra/system.log

Traces
* ./nodetool settraceprobability 0.1
* Saved traces can then be viewed in the system_traces keyspace

Adding a new Node
* The hash ring changes, will nodes move data to next nodes in the ring?
* Talking to seed nodes to understand structure

Node is Down?

Decommission a node?

Tomestone and Performance


Driver Choosing Coordinator
* on a per-query basis
* TokenAwarePolicy driver chooses nodes containing the data
* RoundRobinPolicy
* DCAwareRoundRobinPolicy -?

Which partition resides on which node?
* SELECT token(tag), tag FROM videos_by_tag;
* nodetool getendpoints killrvideo videos_by_tag 'cassandra'

VNode
* https://www.datastax.com/blog/2012/12/virtual-nodes-cassandra-12
* If you want to get started with vnodes on a fresh cluster, however, that is fairly straightforward. Just don't set the initial_token parameter in your conf/cassandra.yaml and instead enable the num_tokens parameter. A good default value for this is 256.


Hint Handoff
* Default to saved for 3 hours in coordinator node
* When CL=ANY, it means a WRITE is successfully as long as written to coordinator, even if NONE of the replicas nodes are running.

Read Path
*

Tabel stats
* /home/ubuntu/node/bin/nodetool cfstats keyspace1.standard1

Memory cache in Read Path
* key cache
* Bloom Filter
* Partition Summary

Stored on Disk in Read Path
* Partition Indexes
* SSTable

Avoid
* Hot partition
  * too active compared with other partitions
* Big Partition
  * up to 2 billion cells per partition
  * up to 100K rows
  * up to 100MB
* Big and constantly growing partition


## My Astra instance created ## 
https://astra.datastax.com/org/bed1dfd5-1d51-41db-8f52-a2fadbba016b/database/501a865d-39cd-4c1d-880e-94c711622005



## Create your own tables on Astra ##

Example tables that we used in the workshop:

```
CREATE TABLE IF NOT EXISTS comments_by_user (
    userid uuid,
    commentid timeuuid,
    videoid uuid,
    comment text,
    PRIMARY KEY ((userid), commentid)
) WITH CLUSTERING ORDER BY (commentid DESC);

CREATE TABLE IF NOT EXISTS comments_by_video (
    videoid   uuid,
    commentid timeuuid,
    userid    uuid,
    comment   text,
    PRIMARY KEY ((videoid), commentid)
) WITH CLUSTERING ORDER BY (commentid DESC);
```

Show us your own tables - for the data model of your choice.

```
Update with your own table
```

## Insert some data ##

Here some example data that we used in the workshop:

```
INSERT INTO comments_by_user (userid, commentid, videoid, comment)
VALUES (11111111-1111-1111-1111-111111111111, NOW(), 12345678-1234-1111-1111-111111111111, 'I keep watching this video');

INSERT INTO comments_by_user (userid, commentid, videoid, comment)
VALUES (11111111-1111-1111-1111-111111111111, NOW(), 12345678-1234-1111-1111-111111111111, 'Soo many comments for the same video');
```

Show off your own data inserts, into your own tables:

```
Your data goes here
```

Now show the output, for example:

```
SELECT * FROM <your table>;
...
...
...
```

Include some screenshots!

## Experiment with CRUD and show the outputs: ##

Examples from the workshop:

```
UPDATE comments_by_video 
SET comment = 'This is fun!' 
WHERE videoid = 12345678-1234-1111-1111-111111111111 AND commentid = 494a3f00-e966-11ea-84bf-83e48ffdc8ac;

SELECT * FROM comments_by_video;
```

```
DELETE FROM comments_by_video 
WHERE videoid = 12345678-1234-1111-1111-111111111111 AND commentid = 494a3f00-e966-11ea-84bf-83e48ffdc8ac;

SELECT * FROM comments_by_video;
```

Show us something similar with your own tables.

Try something different:

Check out the CQL reference and try commands that we did not use in the workshop:

https://docs.datastax.com/en/cql-oss/3.3/cql/cql_reference/cqlReferenceTOC.html

Let us know what you find:

```
Update with your own examples
```

Or connect, read and write to your Astra database via other methods.

Tell us how you do it, we would love to know. 

```
Show your connection code
```

The starry sky is the limit: Build your own app with Astra and show it off for a chance to have it included with our Sample Galleries



