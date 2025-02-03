
## Introduction
- **Apache Kafka** is an event streaming platform used for collecting, processing, storing, and integrating data at scale.
- It has various use cases, including distributed logging, stream processing, data integration, and pub/sub messaging.
- **Events** are actions, incidents, or changes recorded by software or applications, such as payments, website clicks, or temperature readings.
- Events consist of a notification (when-ness) and state, usually represented in a structured format like JSON, Avro, or Protocol Buffers.
- Kafka models events as **key/value pairs**, where keys and values are sequences of bytes. These are often structured objects in your programming language's type system.
- The process of translating between language types and internal bytes is called **serialization and deserialization**.

## Topics

- **Kafka Topics**: The fundamental unit of organization in Kafka, similar to a table in a relational database.
- **Event Organization**: Topics are used to hold different kinds of events and their filtered or transformed versions.
- **Log Structure**: Topics are logs of events, which are append-only, read sequentially, and immutable.
- **Durability**: Logs are durable, with data stored on disk. Topics can be configured to expire data after a certain age or size, or retain messages indefinitely.
- **High Throughput**: The simple semantics of logs enable Kafka to deliver high levels of sustained throughput and make replication easier.

## Partitions

- **Partitioning for Scalability**: Kafka partitions topics to distribute the load across multiple nodes in a cluster, allowing for better scalability and performance.
- **Multiple Logs**: A single topic log is broken into multiple logs (partitions), each residing on a different node.
- **Message Distribution**:
  - **Without Key**: Messages are distributed round-robin among all partitions, ensuring an even share of data but not preserving message order.
  - **With Key**: Messages are assigned to partitions based on a hash of the key, ensuring that messages with the same key always go to the same partition and maintain order.
- **Use Case Example**: Using a customer ID as the key ensures that all events for a specific customer are ordered correctly, even if it results in a more active partition.

## Brokers

- **Kafka Brokers**: These are the machines that make up the Kafka infrastructure.
- **Deployment**: Brokers can be physical servers, containers, or virtualized servers.
- **Function**: Each broker runs the Kafka broker process, hosting partitions and handling requests to write or read events.
- **Replication**: Brokers also manage the replication of partitions between each other to ensure data durability and availability.

## Replication:

- **Replication Necessity**: To ensure data safety, partition data is copied to multiple brokers.
- **Leader and Follower Replicas**: The main partition is the leader replica, and the copies are follower replicas.
- **Replication Process**: Data is produced to the leader, which then replicates the data to the followers automatically.
- **Durability**: Developers can adjust settings for different levels of durability, but generally, the process ensures data safety and availability even if a node fails.

## Producers:

- **KafkaProducer Class**: Used to connect to the Kafka cluster with configuration parameters, including broker addresses and security settings.
- **ProducerRecord Class**: Holds the key-value pair to be sent to the cluster.
- **API Surface**: The producer library manages connection pools, network buffering, message acknowledgments, and retransmissions, simplifying the process for developers.
- **Example Code**: Demonstrates creating a KafkaProducer, sending messages in a loop, and handling exceptions.
```java
  try (KafkaProducer<String, Payment> producer = new KafkaProducer<>(props)) {

    for (long i = 0; i < 10; i++) {
      final String orderId = "id" + Long.toString(i);
      final Payment payment = new Payment(orderId, 1000.00d);
      final ProducerRecord<String, Payment> record = new ProducerRecord<>(
        "transactions", payment.getId().toString(), payment
      );
      producer.send(record);
    }
  } catch (final InterruptedException e) {
    e.printStackTrace();
  }
```
- **Partitioning**: The producer decides which partition to send each message to, either round-robin for keyless messages, by hashing the key, or using a custom scheme.

## Consumers:

Here's a comprehensive summary of Kafka consumers, merging all the key points:

- **KafkaConsumer Class**: Used to connect to the Kafka cluster with configuration parameters, including broker addresses and security settings.
- **Subscription**: Consumers can subscribe to one or more topics, or use a regular expression to match multiple topics.
- **ConsumerRecords**: Messages are received in a collection called ConsumerRecords, containing individual ConsumerRecord objects, which represent the key/value pairs of messages.
- **Cluster Metadata Management**: KafkaConsumer manages connection pooling and keeps up-to-date with cluster metadata, such as node failures and partition reassignments.
- **Message Persistence**: Reading a message does not destroy it, allowing multiple consumers to read from the same topic without any special configuration.
- **Scalability**: Consumers need to handle high message consumption rates and computational costs. Kafka allows scaling consumer groups automatically, distributing messages across multiple instances of the consuming application.
- **Message Ordering**: Within a partition, messages are strictly ordered. However, there is no guarantee of ordering between partitions.
- **Automatic Rebalancing**: When you add or remove a consumer group instance, Kafka automatically rebalances the partitions among the instances. This ensures fair distribution of partitions and maintains scalability and fault tolerance.
- **Group ID**: Each consumer must specify a group ID when created. This group ID is essential for the rebalancing process and scaling consumer groups.
- **Comparison with Traditional Messaging Systems**: Kafka maintains ordering guarantees by key even when scaling out consumer groups, unlike traditional messaging systems where scaling often sacrifices ordering guarantees.
- **Horizontal Scaling**: Kafka's consumer groups allow for horizontal scaling with strong ordering guarantees, making it easier to build scalable and reliable applications.

```java
  try (final KafkaConsumer<String, Payment> consumer = new KafkaConsumer<>(props)){
    consumer.subscribe(Collections.singletonList(TOPIC));

    while (true) {
      ConsumerRecords<String, Payment> records = consumer.poll(100);
      for (ConsumerRecord<String, Payment> record : records) {
        String key = record.key();
        Payment value = record.value();
        System.out.printf("key = %s, value = %s%n", key, value);
      }
    }
  }
```

## Ecosystem:

- **Core Kafka System**: Brokers manage partitioned, replicated topics with producers and consumers writing and reading events, forming a useful system.
- **Common Patterns**: Certain patterns emerge, leading developers to build common layers of application functionality repeatedly.
- **Infrastructure Code**: This code is important but not directly tied to the business value. It should be provided by the community or an infrastructure vendor.
- **Examples**: Kafka Connect, the Confluent Schema Registry, and Kafka Streams are examples of such infrastructure code.

## Kafka Connect:

- **Purpose**: Kafka Connect is Apache Kafka's integration API, used to move data between Kafka topics and external systems.
- **Functionality**: It consists of pluggable connectors and a client application. The client application runs independently of Kafka brokers and can be scaled and made fault-tolerant by running a cluster of Connect workers.
- **Configuration**: Kafka Connect uses JSON configuration to run, abstracting away the need for custom code.
- **Connectors**: 
  - **Source Connectors**: Read data from external systems and produce it to Kafka topics.
  - **Sink Connectors**: Subscribe to Kafka topics and write the messages to external systems.
- **Ecosystem**: Kafka Connect has a large ecosystem of connectors, making it easy to integrate with various systems.

## Confluent Schema Registry:

- **Purpose**: Confluent Schema Registry helps manage the evolving schemas of messages in Kafka topics, ensuring compatibility and understanding across different applications.
- **Functionality**: 
  - **Standalone Server**: Runs independently of Kafka brokers, maintaining a database of schemas.
  - **Schema Database**: Stored in an internal Kafka topic and cached for low-latency access.
  - **High Availability**: Can be configured for redundancy to ensure uptime.
- **API**: Allows producers and consumers to check schema compatibility before producing or consuming messages.
  - **Producers**: Validate new message schemas against previous versions to ensure compatibility.
  - **Consumers**: Ensure they can read messages with the expected schema.

## Kafka Streams

1. **Complexity in Kafka Consumers**  
   - Initially simple consumers evolve into complex tasks like aggregation and enrichment.  
   - The standard Kafka consumer API lacks built-in support for stateful operations like handling time windows, late messages, and aggregations.  

2. **State Management Challenges**  
   - Stateful operations require memory, making fault tolerance difficult.  
   - Without a persistence mechanism, application failures can result in data loss.  

3. **Kafka Streams Overview**  
   - A **Java API** that simplifies stream processing by providing built-in operations like filtering, grouping, aggregation, and joining.  
   - Automatically manages distributed state, persisting data to **local disk** and **Kafka internal topics**.  
   - Allows for **scalable** and **fault-tolerant** stream processing by distributing workload across multiple nodes.  

4. **Integration with Microservices**  
   - Can be integrated into microservices alongside other frameworks (e.g., Spring, Micronaut).  
   - Example: A shipment notification service joins shipment data with customer records while also handling REST API requests.  

5. **Stream API Example**  
   - Demonstrates an application that:  
     - Processes a stream of movie ratings.  
     - Computes an **average rating** per movie.  
     - Joins it with a **movies table** to produce enriched movie data.  
     - Stores results in a new Kafka topic.  
   - Uses **KStream** for event streams and **KTable** for stateful aggregations.  

```java
  StreamsBuilder builder = new StreamsBuilder();

  builder.stream("raw-movies", Consumed.with(Serdes.Long(), Serdes.String())).mapValues(Parser::parseMovie).map((key, movie) -> new KeyValue<>(movie.getMovieId(), movie)).to("movies", Produced.with(Serdes.Long(), movieSerde));

  KTable<Long, Movie> movies = builder.table("movies", Materialized.<Long, Movie, KeyValueStore<Bytes, byte[]>>as("movies-store").withValueSerde(movieSerde).withKeySerde(Serdes.Long()));

  KStream<Long, String> rawRatings = builder.stream("raw-ratings", Consumed.with(Serdes.Long(), Serdes.String()));
  KStream<Long, Rating> ratings = rawRatings.mapValues(Parser::parseRating).map((key, rating) -> new KeyValue<>(rating.getMovieId(), rating));

  KStream<Long, Double> numericRatings = ratings.mapValues(Rating::getRating);

  KGroupedStream<Long, Double> ratingsById = numericRatings.groupByKey();

  KTable<Long, Long> ratingCounts = ratingsById.count();
  KTable<Long, Double> ratingSums = ratingsById.reduce((v1, v2) -> v1 + v2);

  KTable<Long, Double> ratingAverage = ratingSums.join(ratingCounts, (sum, count) -> sum / count.doubleValue(),Materialized.as("average-ratings"));

  ratingAverage.toStream().to("average-ratings");

  KTable<Long, String> ratedMovies = ratingAverage.join(movies, (avg, movie) -> movie.getTitle() + "=" + avg);

  ratedMovies.toStream().to("rated-movies", Produced.with(Serdes.Long(), Serdes.String()));
```

6. **Further Learning**  
   - Kafka Streams 101 course and Michael Noll’s series on **Streams and Tables in Apache Kafka** are recommended for a deeper understanding.