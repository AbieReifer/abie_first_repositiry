# Kafka #

## Installing Kafka

Kafka is installed via Ambari. It exists in /usr/hdp/2.5.3.0-37/kafka, with scripts residing in the /bin folder.

## Kafka Monitoring

The HDP platform includes Ambari Metrics which captures a number of Kafka metrics and displays them in Ambari or Grafana.

## Configuring Kafka
Most configuration is done via Ambari, with the exception of ocnfiguring topics, see below.

We are generally following, to the extent hardware resources permit, the HDP Best Practices guide for Kafka.
See https://community.hortonworks.com/articles/80813/kafka-best-practices-1.html 

### JVM
JVM parameters are set in the ```KAFKA_HEAP_OPTS``` system variable, which can be edited in ```kafka-env template``` in Ambari
```
export KAFKA_HEAP_OPTS="-Xmx8g -Xms8g
 -XX:MetaspaceSize=96m
 -XX:+UseG1GC
 -XX:MaxGCPauseMillis=20 
 -XX:InitiatingHeapOccupancyPercent=35
 -XX:G1HeapRegionSize=16M
 -XX:MinMetaspaceFreeRatio=50
 -XX:MaxMetaspaceFreeRatio=80"
```

## Configuring Topics

Topics are configured using the kafka-topics.sh script, which is installed in the same location as the Kafka binaries.  

For example, in the Test environment, Kafka is installed on hdp-c21 and Kafka binaries are at /usr/hdp/current/kafka-broker/bin/

### List topics
```
cd /usr/hdp/current/kafka-broker/bin
./kafka-topics.sh --zookeeper localhost:2181 --list
```

### Describe a topic
Shows number of partitions in use.
```
cd /usr/hdp/current/kafka-broker/bin
./kafka-topics.sh --zookeeper localhost:2181 --describe --topic "TOPIC_NAME"
```

### Delete a topic
Kafka config has to have ```delete.topic.enable=true```, this can be done via Ambari.
```
cd /usr/hdp/current/kafka-broker/bin
 ./kafka-topics.sh --zookeeper localhost:2181 --delete --topic "TOPIC_NAME"
```

### Configure Number of Partitions

You are advised to stop all processes that are publishing/consuming Kafka topics before you modify partitions. 
Kafka partition counts can only be increased, otherwise you'll have to drop and recreate them. 
If you choose not to stop the processes before altering the topic, the ordering of messages cannot be guaranteed.  

For best performance, ensure the number of partitions you create is evenly divisible among the number of consumer threads you expect to have.

For example, in the NHCS project the application.properties file includes some settings for Kafka.  
```kafka.concurrency=10```, this is the number of threads to create for each consumer.  

If concurrency = 10, and you expect to deploy two instances of the service, you should have at least 20 partitions.
You do this through the --alter comand, as below.
```
cd /usr/hdp/current/kafka-broker/bin
./kafka-topics.sh --zookeeper localhost:2181 --topic "TOPIC-NAME" --alter --partitions 20
```
For example, for the NHCS project the number of partitions is set for each topic based on the default configuration (concurrency) for each service. 

This assumes;
- nhcs-validation and nhcs-storage concurrency=30 and 2 instances are deployed. 
- all other services, concurrency=10, and 1 instance is deployed (2 instances could be deployed).

```
./kafka-topics.sh --zookeeper localhost:2181 --topic "nhcs-log-log" --alter --partitions 20
./kafka-topics.sh --zookeeper localhost:2181 --topic "nhcs-notification-event" --alter --partitions 20
./kafka-topics.sh --zookeeper localhost:2181 --topic "nhcs-receipt-notification" --alter --partitions 20
./kafka-topics.sh --zookeeper localhost:2181 --topic "nhcs-storage-txt" --alter --partitions 60
./kafka-topics.sh --zookeeper localhost:2181 --topic "nhcs-storage-xml" --alter --partitions 60
./kafka-topics.sh --zookeeper localhost:2181 --topic "nhcs-text-received" --alter --partitions 20
./kafka-topics.sh --zookeeper localhost:2181 --topic "nhcs-validation-txt" --alter --partitions 60
./kafka-topics.sh --zookeeper localhost:2181 --topic "nhcs-validation-xml" --alter --partitions 60
```
