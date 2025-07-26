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