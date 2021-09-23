# Apache Kafka, Docker & Zookeeper
A 3 node Apache Kafka cluster using Docker that demonstrates Pub-Sub messaging, with a Zookeeper ensemble to manage the Kafka cluster.

## Prerequisites
Ensure Docker is installed & running:

```
docker -v
> Docker version 19.03.12, build 48a66213fe
```

as well as Docker Compose:

```
docker-compose -v
> docker-compose version 1.26.2, build eefe0d31
```

## 1. Start the cluster

```
docker-compose up -d
```

## 2. Test the cluster (Pub-Sub Messaging)
### 2.1 Create a new topic 
Create a new topic called *mytopic*.

```
docker exec -it kafka1 kafka-topics.sh --create --topic mytopic --bootstrap-server kafka1:9092, kafka2:9092, kafka3:9092
```
### 2.2 Set up a message consumer
Start a new message consumer using one of the nodes (kafka1, kafka2 or kafka3). This will be where the messages will be recieved, as they are published to *mytopic*.
```
docker exec -it kafka2 kafka-console-consumer.sh --topic mytopic --bootstrap-server kafka1:9092, kafka2:9092, kafka3:9092
```
### 2.3 Set up a message producer
In a __new terminal shell__, set up a message producer using a different node, by entering the command below and sending a few messages - each on a separate line.
```
docker exec -it kafka3 kafka-console-producer.sh --topic mytopic --bootstrap-server kafka1:9092, kafka2:9092, kafka3:9092
> hi there
> this is a message
> this is another message
```
### 2.4 Verify if messages have been recieved
Return to the previous terminal shell (of the message consumer) to see if the messages have been recieved. They should be displayed right below the command.

```
docker exec -it kafka2 kafka-console-consumer.sh --topic mytopic --bootstrap-server kafka1:9092, kafka2:9092, kafka3:9092

hi there
this is a message
this is another message
```

## Managing Zookeeper failover
According to the [HBase documentation](http://hbase.apache.org/book.html#zookeeper), it is recommended that you run a ZooKeeper ensemble of 3, 5 or 7 machines; the more members an ensemble has, the more tolerant the ensemble is of host failures. In this project, I've used 3 ZooKeepers for failover management. As such, if one fails, another ZooKeeper would automatically take over as the leader. We can replicate this by
finding the current leader,
```
docker exec -i -t <zk1/zk2/zk3> zkServer.sh status
```
terminating it,
```
docker kill <zk1/zk2/zk3>
```
and verifying that another ZooKeeper has taken over as leader (by repeating the first step of this section).
