# Kafka

## Overview

Kafka is a streaming platform has three key capabilities:

<ul>
<ii>Publish and subscribe to streams of records, similar to a message queue or enterprise messaging system</ii>.
<li>Store streams of records in a fault-tolerant durable way.</li>
<li>Process streams of records as they occur.</li>
</ul>

Kafka is generally used for two broad classes of applications:
<ul>
<li>Building real-time streaming data pipelines that reliably get data between systems or applications </li>
<li>Building real-time streaming applications that transform or react to the streams of data</li>
</ul>

![Kafka overview](/images/Overview.png)

## Kafka Theory

### Topics Partitions and offsets

Topic is particular stream of data, it is a category or feed name to which records are published. it is similar to a 
table in database. You can have many topics as you want. A topic is identified by its name.

For each topic kafka maintains  a partitioned log like this

![Partitions in Topics](/images/Partitions.png)

<p>You need to specify the number of partitions while creating a topic.Each partition is ordered, immutable sequence of 
records.</p>

<p>Each sequence of record has unique id called as offsets
offsets uniquely identifies each record within partitions.
Kafka clusters preserve records for short period of time which can be configured
after the short period of time they are no longer available for subscribers to consume.

<p><b>Note</b> the only metadata retained on a per-consumer basis is the offset or position of that consumer in the log. 
This offset is controlled by the consumer: normally a consumer will advance its offset linearly as it reads records, 
but, in fact, since the position is controlled by the consumer it can consume records in any order it likes. 
For example a consumer can reset to an older offset to reprocess data from the past or skip ahead to the most recent 
record and start consuming from "now".</p>

### Brokers

A kafka cluster is composed of multiple brokers (Servers). Each Broker is identified with its ID(integer)
Each broker  contains certain topic partitions and after connecting to any broker called a bootstrap broker you will be 
connected to entire cluster. 

<p><b>Topics and Brokers</b></p>

<p>When we create a topic with Kafka, it will distribute the partitions for the topic across all brokers/servers</p>

### Replication and Distribution

<p>
At any time, one broker can be leader for a given partition. Only that leader can receive and serve data for a 
partition. THe other brokers would synchronize the data. Therefore, each partition has one leader and multiple 
in - sync replica
</p>

<p>
In kafka cluster, you need to specify the replication factor for each cluster that will replicate partitions for
various topics on other brokers.
</p>

<p>Zookeeper decides who will be the leader for given partition</p>

### Producers

<p>Producers write data to topics. Producers automatically know to which broker and partition to write to, Inc case of 
Broker failures, producers will automatically recover. A producer can send data to multiple partitions</p>

<p>Producer can choose to receive the acknowledgement of data writes
</p>

<p>Producer can choose to send a key within the message. if the key =null then data is sent Round Robin. if a key is 
sent then all messages for that key will always go to the same partition. A key is basically sent if you need message 
ordering for a specific field</p>

### Consumers & Consumers Group

Consumers read data from a topic identified by name. Consumers know which broker to read from. In case of broker 
failures, consumers to know how to recover Data is read within each partitions.

<p>
Consumers read data in consumer groups. Each consumer within a group reads from exclusive partitions. if you have more 
consumers than partitions in  a group, some consumers will be inactive. if you have less consumers than partition then 
some consumer will read from multiple partitions.

<b>Note: Each consumer will automatically know to read from which partition. This is taken care by GroupCoordinator and
 and a consumerCoordinator</b>
</p>

### Consumer offsets

<p>Kafka stores the offset at which the consumer group is reading . These offsets are committed live in Kafka topic 
named __consumer_topic. when a consumer in a group has processed data received from Kafka, it should be committing thw 
offsets. when a consumer dies and comes back, it will be able to read back from where it left off, Thanks to the 
committed offsets.
</p>

<p>Consumers choose when to commit offsets. There are 3 delivery semantics
once when the message is received, once when the message is processed. Exactly once using kafka to kafka 
workflows.

<b>Note: use idempotent consumer to avoid duplicates in the database or Kafka processing</b>
</p>
 
### Kafka broker discovery

<p>Every Kafka broker is called bootsreap server, that means that you only need to connect to one broker and you will be 
connected to the entire cluster. Each broker knows about all brokers, topics and partitions (metadata)</p>