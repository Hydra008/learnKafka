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

### zookeeper

zookeeper manages brokers and maintains a list of them. Zookeeper helps in performing leader election for partitions.
Zookeeper sends notification to kafka in case of changes in new topic, broker dies, broker comes up, deletion of topic 
and so on

<p>Kafka can't work without zookeeper. zookeeper always operates on odd number of servers</p>

### Kafka Guarantees

<ul>
<li>Messages are appended to a topic-partition in the order they are sent</li>
<li>Consumers read messages in the order stored in topic partition</li>
<li>With a replication factor of N, producers and consumers tolerate upto N-1 brokers bring down</li>
</ul>

### Kafka Architecture

[!Kafka Architecture](/images/Kafka%20Architectire.png)


## Installing and Starting Kafka

To install Kafka on MacOS

```
brew install kafka
```

To run Kafka on MacOs

```
zkserver start
```

Alternatively
```
cd /usr/local/Cellar/kafka/2.3.1/libexec/
bin/zookeeper-server-start.sh config/zookeeper.properties
```

To configure zookeeper properties, change  data default zookeeper directory
```
cat config/zookeeper.properties
mkdir data
mkdir data/zookeeper
nano /usr/local/Cellar/kafka/2.3.1/libexec/config/zookeeper.properties
```
change dataDir to whatever you want

<p>
To Change Kafka data directory</p>

```
nano /usr/local/Cellar/kafka/2.3.1/libexec/config/server.properties
```

Starting Kafka server
```
kafka-server-start config/server.properties
```

## Kafka CLI 

### Creating a topic

To create a topic you need to specify the topic name, number of partitions and replica sets. YOu will also need to 
specify the url of zookeeper

```a
kafka-topics --zookeeper 127.0.0.1:2181 --topic firstTopic --create --partition 3 -- replication-factor 1
```

<p>Also Note you cannot have more replication factors then the number of zookeeper brokers</p>

### List of topicss or topics in detail

```
kafka-topics --zookeeper 127.0.0.1:2181 --list
```

<p>To get more information on specific topic use the following command</p>

```
kafka-topics --zookeeper 127.0.0.1:2181 --topic firstName --describe
```

### Delete a topic

To delete a Topic,

```
kafka-topics --zookeeper 127.0.0.1:2181 --topic firstName --delete
```

<p>Also note that in server.properties delete.topic.enable is set to true for deletion.
</p>

### producing to Kafka topic

<p>Also, you can get acknowledgement
   for the data being sent by using --producer-property acks=all as follows</p>

``` 
kafka-console-producer --broker-list 127.0.0.1:90--topic first_topic --producer-property acks=all
```
### Creating a topic that never existed before

<p>when we create a topic that never existed before, we will see a warning, that says no leader available, However in few 
seconds a leader will be elected and a new topic will be created with default partition and replication factor of 1</p>

<p>One can change the default in server.properties</p>

### Consuming a Kafka topic 

<p>To consume a Kafka topic we need to specify the following command</p>

```
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic firstTopic
```

The above command will only receive live messages produced by producers. To receive all messages from beginning we can
add the following flag to command

```
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic firstTopic --from-beginning
```

### Consuming a Kafka topic in a consumer group

You can use the --group flag to specify the consumer group like this in multiple terminal
windows. This will enable data sent by Kafka producer to be consumed by multiple consumers
some data can be sent to consumer 1 & some to consumer 2. it depends on the number of consumers in the group
to the total number of partitions in the group.

``` 
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --group myApp
```

<p>Also you can use --describe command to fetch additional information about the consumers in the consumer group</p>

``` 
kafka-consumer-groups --bootstrap-server 127.0.0.1:9092  --group myApp --describe
```

### Reading from a topic at different offset & resetting offsets

To let consumer read a message from the start in the topic, we can use the following command

```
kafka-consumer-groups --bootstrap-server 127.0.0.1:9092  --group myApp --reset-offsets --to-earliest --execute 
--topic first_topic
```

## Kafka Schema Registry & Avro

### Why do we need Schema is in Kafka

<p>Kafka takes binary input and distributes the binary to consumers. it does not even know if the data is string or int.
So if someone sends bad data, or field changes the consumers will break. Hence we would need to maintain the schema 
outside of kafka to avoid any negative performance of the cluster.. This is where schema registry comes in place
and apache avro is just the data format 
</p>


### Avro

Avro is defined by a schema written JSON. to simplify, you can view Avro as JSON with schema attached to it. Avro 
compresses the data it does not have support for all languages. You can embed the documentation in the Avro itself

<p>
Avro has support for major primitive types such as bull, boolean, int, long, float, double, bytes, string. A file 
with extension.avsc  is avro schema file. 
</p>

<b>Avro Record Schemas</b> 

<p>it has some common fields</p>

<ul>
    <li>Name: Name of your schema</li>
    <li>Namespace: Equivalent of package in Java</li>
    <li>Doc: Documentation to explain your schema</li>
    <li>Aliases: Optional other Names of your schema</li>
    <li>Fields 
     <ul>
        <li> Name: Name of your field</li>
        <li> Doc: Documentation of that field</li>
        <li> Type: Data Type of that field</li>
        <li> Default: Default value of your field</li>
     </ul>
     </li>
</ul>

<p>Avro has support for complex types such as
Enums,Arrays, Maps, Unions, Calling other schemas as types
</p>

<b>Enum</b>

These are fields you know for sure that their values can be enumerated

Example: Customer status (Bronze, Silver, Gold)

<b>Once enum is set, changing the values if forbidden to maintain compati bility</b>

<b>Arrays</b>

Example: {type:Array, items: "string"}

<b>Maps</b>

Example: {type: "map", values: "string"}

<b>Unions</b>

Unions can allow a field to take different types

Example ["string", "int", "boolean"]

Use case: to define optional values as follows
{"name" : "middle_name", "type": ["null","string"], default: null}

<b>Avro: Logical Types</b>

Avro has support for many logical types such as decimals (bytes), date(int), time-millis, timestamp-millis(long)

Example: {name: "signup_ts", type: "long", logicalType: "timestamp-millis"}

## Avro in Java

### Generic Record

<p>A GenericRecord is used to create an avro object from schema, the schema referenced here can be a</p>
<ul>
    <li>File</li>
    <li> A string </li>
</ul>

it is not the most recommended way of creating Avro objects because it can fail at runtime. it is the most simplest way

```java

import org.apache.avro.Schema;
import org.apache.avro.generic.*;

        Schema.Parser parser = new Schema.Parser();
        Schema schema = parser.parse("{\n" +
                "     \"type\": \"record\",\n" +
                "     \"namespace\": \"com.example\",\n" +
                "     \"name\": \"Customer\",\n" +
                "     \"doc\": \"Avro Schema for our Customer\",     \n" +
                "     \"fields\": [\n" +
                "       { \"name\": \"first_name\", \"type\": \"string\", \"doc\": \"First Name of Customer\" },\n" +
                "       { \"name\": \"last_name\", \"type\": \"string\", \"doc\": \"Last Name of Customer\" },\n" +
                "       { \"name\": \"age\", \"type\": \"int\", \"doc\": \"Age at the time of registration\" },\n" +
                "       { \"name\": \"height\", \"type\": \"float\", \"doc\": \"Height at the time of registration in cm\" },\n" +
                "       { \"name\": \"weight\", \"type\": \"float\", \"doc\": \"Weight at the time of registration in kg\" },\n" +
                "       { \"name\": \"automated_email\", \"type\": \"boolean\", \"default\": true, \"doc\": \"Field indicating if the user is enrolled in marketing emails\" }\n" +
                "     ]\n" +
                "}");


        // create a generic record
        GenericRecordBuilder customerBuilder = new GenericRecordBuilder(schema);
        customerBuilder.set("first_name", "John");
        customerBuilder.set("last_name", "Doe");
        customerBuilder.set("age", 26);
        customerBuilder.set("height", 170f);
        customerBuilder.set("weight", 80.5f);
        customerBuilder.set("automated_email", false);

        GenericData.Record myCustomer = customerBuilder.build();
        System.out.println(myCustomer);

```

### Specific Record

In this method of writing Avro records, we need to follow the following steps. it is a recommended way of generating 
avro records because it does not give runtime errors. 

First, we need to store our .avsc file in src/main/resources/avro/*.avdc

then in pom.xml, we need to add following snippet to generate Java object class from the avro

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>


            <!--for specific record-->
            <plugin>
                <groupId>org.apache.avro</groupId>
                <artifactId>avro-maven-plugin</artifactId>
                <version>${avro.version}</version>
                <executions>
                    <execution>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>schema</goal>
                            <goal>protocol</goal>
                            <goal>idl-protocol</goal>
                        </goals>
                        <configuration>
                            <sourceDirectory>${project.basedir}/src/main/resources/avro</sourceDirectory>
                            <stringType>String</stringType>
                            <createSetters>false</createSetters>
                            <enableDecimalLogicalType>true</enableDecimalLogicalType>
                            <fieldVisibility>private</fieldVisibility>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <!--force discovery of generated classes-->
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>build-helper-maven-plugin</artifactId>
                <version>3.0.0</version>
                <executions>
                    <execution>
                        <id>add-source</id>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>add-source</goal>
                        </goals>
                        <configuration>
                            <sources>
                                <source>target/generated-sources/avro</source>
                            </sources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

Now, go to Maven Lifecycle, Click on clean and then package. This will make a clean build and package your project. 
In your target/generated-sources/avro You will have the Java Class created for schema which is easy to use.

```Java
package com.github.hydra008.kafka;

import com.example.Customer;
import org.apache.avro.file.DataFileReader;
import org.apache.avro.file.DataFileWriter;
import org.apache.avro.io.DatumReader;
import org.apache.avro.io.DatumWriter;
import org.apache.avro.specific.SpecificDatumReader;
import org.apache.avro.specific.SpecificDatumWriter;

import java.io.File;
import java.io.IOException;

public class SpecificRecordExample {
    public static void main(String[] args) {

        // create specific record
        Customer.Builder customerBuilder = Customer.newBuilder();
        customerBuilder.setAge(30);
        customerBuilder.setFirstName("Mark");
        customerBuilder.setLastName("Simpson");
        customerBuilder.setAutomatedEmail(true);
        customerBuilder.setHeight(180f);
        customerBuilder.setWeight(90f);

        Customer customer = customerBuilder.build();
        System.out.println(customer.toString());

        // write it out to a file
        final DatumWriter<Customer> datumWriter = new SpecificDatumWriter<>(Customer.class);

        try (DataFileWriter<Customer> dataFileWriter = new DataFileWriter<>(datumWriter)) {
            dataFileWriter.create(customer.getSchema(), new File("customer-specific.avro"));
            dataFileWriter.append(customer);
            System.out.println("successfully wrote customer-specific.avro");
        } catch (IOException e){
            e.printStackTrace();
        }


        // read it from a file
        final File file = new File("customer-specific.avro");
        final DatumReader<Customer> datumReader = new SpecificDatumReader<>(Customer.class);
        final DataFileReader<Customer> dataFileReader;
        try {
            System.out.println("Reading our specific record");
            dataFileReader = new DataFileReader<>(file, datumReader);
            while (dataFileReader.hasNext()) {
                Customer readCustomer = dataFileReader.next();
                System.out.println(readCustomer.toString());
                System.out.println("First name: " + readCustomer.getFirstName());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        //interpret
    }
}

```

### Reflections

Reflections help to create an Avro Schema form an existing Java Class

### Schema Evolution

Avro Enables to evolve schema over a time to adapt to business needs. There are 4 kinds of Schema Evolution

#### Backward

Backwards: New Schema can be used with old data <br>
Example
<table>
    <tr>
        <th>Schema 1 </th>
        <th>Schema 2</th>
    </tr>
        <tr>
            <td>fname,lname </td>
            <td>fname,lname, phonNumber(Default: "000-000-0000")</td>
        </tr>
</table>

The new schema can let us read old data fname and lname as default is set for phoneNumber

#### Forward

Forward: Old Schema can be used to read new data

<table>
    <tr>
        <th>Schema 1 </th>
        <th>Schema 2</th>
    </tr>
        <tr>
            <td>fname,lname </td>
            <td>fname,lname, phonNumber</td>
        </tr>
</table>

We can read new data with old schema. Avro will just ignore new fields

#### Full
Full: which is backward and forward

<table>
    <tr>
        <th>Schema 1 </th>
        <th>Schema 2</th>
    </tr>
        <tr>
            <td>fname,lname </td>
            <td>fname,lname, phonNumber(Default: "000-000-0000")</td>
        </tr>
</table>

#### Breaking
Breaking: which will break the programs of consumers

Following changes can break things

<ul>
    <li>Adding/Removing elements from enum</li>
    <li>Changing type of an field</li>
    <li>Renaming a required field (field without default)</li>
</ul>

#### Guidelines to write an Evolving Schema

<ul>
    <li>Make Primary Key Default</li>
    <li>Give default values to all fields that could be removed in future</li>
    <li>Enums can't evolve over time</li>
    <li>Don't rename fields. Youc an add aliases instead</li>
    <li>When evolving schema always give default values</li>
    <li>When evolving a schema, never delete a required field</li>
</ul>

### Kafka Schema Registry

#### Purpose of Confluent Schema

<ul>
    <li>Store and retrieve schemas for Producers and Consumers </li>
    <li>Enforce Backward/ Forward/ Full Compatibility on topics</li>
    <li>Decrease the size of payload of data sent to kafka </li>
</ul> 

#### Schema Registry operations

Schema registry allows you add, retrieve, update, delete a schema through a REST API.
Schemas can be applied to both keys and values.

#### Avro producers and consumers

Confluent Kafka comes with kafka avro producers and consumers which are good for testing/debugging a single message.

We can run the following command to generate a topic, register a schema and produce a single message
```
kafka-avro-console-producer \
    --broker-list 127.0.0.1:9092 --topic test-avro \
    --property schema.registry.url=http://127.0.0.1:8081 \
    --property value.schema='{"type":"record","name":"myrecord","fields":[{"name":"f1","type":"string"}]}'
```

The above command will start the producer, we can send messages for example
```
{"f1": "value1"}
```

now, run the following command, to consume the data
```
kafka-avro-console-consumer --topic test-avro \
    --bootstrap-server 127.0.0.1:9092 \
    --property schema.registry.url=http://127.0.0.1:8081 \
    --from-beginning
``` 


