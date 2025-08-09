What is Kafka?
    Apache Kafka is a distributed event streaming platform used for building real-time data pipelines and streaming applications. It’s designed to handle high-throughput, low-latency data feeds. Kafka was originally developed by LinkedIn and is now an open-source project maintained by the Apache Software Foundation.

🔧 Core Concepts
    1. Producer
        Sends (publishes) data to Kafka topics.
        Example: A web app sending user activity logs.

    2. Consumer
        Reads (subscribes to) data from Kafka topics.
        Example: An analytics system processing user behavior.

    3. Broker
        A Kafka server that stores and serves data.
        Kafka clusters typically consist of multiple brokers for scalability and fault tolerance.

    4. Topic
        A named stream of data.
        Data is split into partitions within a topic to allow parallel processing.

    5. Partition
        A topic is split into one or more partitions to distribute load.
        Each partition is an ordered, immutable sequence of messages.

    6. Offset
        A unique identifier for each message within a partition.
        Allows consumers to track what they’ve already read.


🧩 Why Use Kafka?
    High throughput (millions of messages per second)
    Fault tolerance (via replication)
    Scalability (horizontally scalable)
    Durability (persistent storage)
    Real-time processing (low latency)

📦 Common Use Cases
    Logging and monitoring pipelines (e.g., sending logs to Elasticsearch)
    Real-time analytics
    Event-driven architectures
    Data ingestion from IoT devices
    Stream processing with tools like Apache Flink or Kafka Streams

How to start Kafka in local?
🐳 Step-by-Step: Kafka with Docker Compose
1. Install Docker & Docker Compose

2. Create a docker-compose.yml File

    Example:
        services:
            zoo1:
                image: confluentinc/cp-zookeeper:7.3.2
                hostname: zoo1
                container_name: zoo1
                ports:
                - "2181:2181"
                environment:
                ZOOKEEPER_CLIENT_PORT: 2181
                ZOOKEEPER_SERVER_ID: 1
                ZOOKEEPER_SERVERS: zoo1:2888:3888


            kafka1:
                image: confluentinc/cp-kafka:7.3.2
                hostname: kafka1
                container_name: kafka1
                ports:
                - "9092:9092"
                - "29092:29092"
                environment:
                KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka1:19092,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9092,DOCKER://host.docker.internal:29092
                KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT,DOCKER:PLAINTEXT
                KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
                KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
                KAFKA_BROKER_ID: 1
                KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
                KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
                KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
                depends_on:
                - zoo1

3. Start Kafka and Zookeeper
    docker-compose up

4. Test Kafka (Using Kafka CLI inside Docker)
    Open an interactive shell in the Kafka container:
        docker exec -it kafka bash
    Create a topic:
        kafka-topics --bootstrap-server kafka1:19092 --create --topic test-topic --replication-factor 1 --partitions 1
    Start a producer:
        docker exec --interactive --tty kafka1  kafka-console-producer --bootstrap-server kafka1:19092 --topic test-topic
    In another shell, start a consumer:
        docker exec --interactive --tty kafka1  kafka-console-consumer --bootstrap-server kafka1:19092 --topic test-topic --from-beginning

5. Stop Kafka
    docker-compose down


6. Producing and consuming meesage with key-value
    Create a producer:
        docker exec --interactive --tty kafka1  kafka-console-producer --bootstrap-server kafka1:19092 --topic test-topic --property "key.separator=-" --property "parse.key=true"
    Create a consumer:
        docker exec --interactive --tty kafka1  kafka-console-consumer --bootstrap-server kafka1:19092 --topic test-topic --from-beginning --property "key.separator= - " --property "print.key=true"
    
    Now we will be able to see messages with key-value pair seperated by -


Kafka consumer offset
    - In Apache Kafka, a consumer offset refers to the position of a consumer in a Kafka topic partition. It indicates the last record the consumer has successfully read. Offsets are crucial for tracking progress and ensuring reliable consumption of messages.
    - In Kafka, when consuming data from a topic using command-line tools or configuring a consumer client, there are several ways to control from which offset the consumer starts reading.

🔁 Kafka Consumer Offset Commands

1. Consume from the Beginning
This tells the consumer to start reading from the earliest offset (i.e., from the beginning of the topic).
    kafka-console-consumer.sh --bootstrap-server <broker-host>:9092 --topic <topic-name> --from-beginning

2. Consume from the Latest Offset
This starts consuming only new messages (i.e., messages written after the consumer starts).
This is the default behavior unless the topic has no committed offset and auto.offset.reset is set to earliest.


Kafka Client Config (in code or properties):
    auto.offset.reset=latest


3. Consume from a Specific Offset
This requires using consumer groups + offset manipulation via the kafka-consumer-groups.sh tool.

Step 1: Assign a specific offset
    kafka-consumer-groups.sh --bootstrap-server <broker-host>:9092 --group <consumer-group-name> --topic <topic-name> -- <partition-number> --reset-offsets --to-offset <offset-number> --execute



Kafka Consumer group
🔹 What is a Kafka Consumer Group?
A consumer group is a group of one or more consumer instances that work together to consume messages from one or more Kafka topic partitions.
- Each message in a partition is only read by one consumer in the group, enabling parallel consumption without duplication.

🔧 How It Works
- Each partition in a topic is assigned to one consumer within a group.
- Consumers in different groups will all receive the same messages (i.e., parallel, independent consumption).
- Kafka tracks the offsets per consumer group to know what’s been read.

📌 Example Scenario
    Topic: logs with 3 partitions
    Consumer Group: log-processors

    Partition	Consumer	    Group
    0	            C1  	log-processors
    1	            C2	    log-processors
    2	            C3	    log-processors

    Each consumer in the group processes one partition.
    If one consumer dies, Kafka will rebalance and assign its partitions to other consumers.




----------------------------------------------------------------------------------------------------------------------------------------
Docker Compose setup for running a Kafka cluster with 3 brokers (using ZooKeeper). It’s ideal for local development or testing environments.

    version: '2.1'

services:
  zoo1:
    image: confluentinc/cp-zookeeper:7.3.2
    platform: linux/amd64
    hostname: zoo1
    container_name: zoo1
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_SERVER_ID: 1
      ZOOKEEPER_SERVERS: zoo1:2888:3888


  kafka1:
    image: confluentinc/cp-kafka:7.3.2
    platform: linux/amd64
    hostname: kafka1
    container_name: kafka1
    ports:
      - "9092:9092"
      - "29092:29092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka1:19092,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9092,DOCKER://host.docker.internal:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT,DOCKER:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
      KAFKA_BROKER_ID: 1
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
    depends_on:
      - zoo1

  kafka2:
    image: confluentinc/cp-kafka:7.3.2
    platform: linux/amd64
    hostname: kafka2
    container_name: kafka2
    ports:
      - "9093:9093"
      - "29093:29093"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka2:19093,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9093,DOCKER://host.docker.internal:29093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT,DOCKER:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
      KAFKA_BROKER_ID: 2
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
    depends_on:
      - zoo1


  kafka3:
    image: confluentinc/cp-kafka:7.3.2
    platform: linux/amd64
    hostname: kafka3
    container_name: kafka3
    ports:
      - "9094:9094"
      - "29094:29094"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka3:19094,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9094,DOCKER://host.docker.internal:29094
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT,DOCKER:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
      KAFKA_BROKER_ID: 3
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
    depends_on:
      - zoo1

🚀 Startup 
Above containt is added in docker-compse-mutli-broker.yml file
Run docker compse
docker-compose -f docker-compose-multi-broker.yml up

🧪 Example: Create a Topic with Replication
docker exec --interactive --tty kafka1  kafka-topics --bootstrap-server kafka1:19092 --create --topic test-topic --replication-factor 3 --partitions 3

🧾 2. Produce Messages to the Topic
docker exec --interactive --tty kafka1  kafka-console-producer --bootstrap-server kafka1:19092 --topic test-topic

🔄 Consume messages from the topic
docker exec --interactive --tty kafka1  kafka-console-consumer --bootstrap-server kafka1:19092 --topic test-topic --from-beginning



⚙️ Kafka Request Distribution: Core Principles
🔸 1. Cluster Metadata Awareness
    Kafka clients (producers/consumers) fetch cluster metadata from a broker, which includes:
    -List of all brokers
    -Partition information for each topic
    -Leader broker for each partition

👉 Clients become aware of which broker is the leader for each partition, and they direct their requests accordingly.

📤 Producer Request Distribution
    When a producer sends a message:
    1. Partition Selection:
        If a key is provided → Kafka hashes the key to pick a partition.
        If no key → Kafka uses round-robin or sticky partitioning.
    2. Leader Broker Lookup:
        The producer looks up which broker is the leader for that partition (from metadata).
    3. Request Sent:
        The request (produce message) is sent directly to the leader broker for that partition.

📥 Consumer Request Distribution
    When a consumer consumes messages:
    1.Joins a consumer group (via group coordinator).
    2. Kafka assigns partitions to consumers within that group.
    3. The consumer fetches messages directly from the leader broker for each assigned partition.

✅ So each consumer in a group fetches only the partitions it owns, directly from their leader brokers.

🛠️ Broker Roles in Requests
Broker Role     	Description
Leader	            Handles all read/write requests for a partition
Follower	        Replicates data from leader, no client requests
Controller      	Special broker that manages partition leadership and broker metadata

🗺️ Example: Topic with 3 Partitions and 3 Brokers
Partition	Leader Broker	Producer Request	Consumer Request
0	        Broker 1	    Sent to Broker 1	Read from Broker 1
1       	Broker 2	    Sent to Broker 2	Read from Broker 2
2	        Broker 3	    Sent to Broker 3	Read from Broker 3

Even if a client connects to Broker 1 initially, the Kafka client will reroute requests to the correct broker based on metadata.



🔁 Kafka Replication
✅ What is Replication?
Kafka replicates topic partitions across multiple brokers to protect against broker failure and data loss.

🧱 Key Concepts:
Leader Replica: The broker responsible for all reads and writes for a partition.

Follower Replicas: Other brokers that copy data from the leader.

📦 Example:
Topic orders with partition 0, replication factor = 3

Broker	Replica Role
B1	    Leader
B2	    Follower
B3	    Follower

When a producer writes to partition 0, it goes to Broker 1, and then the data is replicated to Brokers 2 and 3.

🔄 In-Sync Replicas (ISR)
🔍 What is ISR?
The ISR is the list of replicas that are fully caught up with the leader — i.e., they have replicated all messages successfully.
Only ISRs are eligible to become the new leader if the current leader fails.

✔️ How ISR Works:
When a message is produced to the leader, Kafka waits for replication to all ISRs (if acks=all is set).
If a follower falls behind (e.g., slow or disconnected), it is removed from the ISR.

🧪 Example ISR Set:
Partition 0, replication factor = 3
Leader: Broker 1
Follower (in-sync): Broker 2
Follower (out-of-sync): Broker 3 (lagging or crashed)

Then ISR = [Broker 1, Broker 2]
















****************************Kafka Commands**************************************************
# List topics in a Kafka cluster
docker exec --interactive --tty kafka1  kafka-topics --bootstrap-server kafka1:19092 --list

# List consumer gropus
docker exec --interactive --tty kafka1 kafka-consumer-groups --bootstrap-server kafka1:19092 --list

# Consume message using consumer group
docker exec --interactive --tty kafka1  kafka-console-consumer --bootstrap-server kafka1:19092 --topic test-topic --group console-consumer-64287 --property "key.separator= - " --property "print.key=true"

# Command to describe all the Kafka topics.
docker exec --interactive --tty kafka1 kafka-topics --bootstrap-server kafka1:19092 --describe

# Command to describe a specific Kafka topic.
docker exec --interactive --tty kafka1  kafka-topics --bootstrap-server kafka1:19092 --describe --topic test-topic

# Alter Kafka partions
docker exec --interactive --tty kafka1  kafka-topics --bootstrap-server kafka1:19092 --alter --topic test-topic --partitions 2

### Log file and related config
- Log into the container.
    docker exec -it kafka1 bash
- The config file is present in the below path.
    /etc/kafka/server.properties
- The log file is present in the below path.
    /var/lib/kafka/data/