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

