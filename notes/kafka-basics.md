source: https://developer.confluent.io/courses/apache-kafka/events/

## What Is Apache Kafka?
[Apache Kafka](https://www.confluent.io/what-is-apache-kafka/?session_ref=https%3A%2F%2Fhub.docker.com%2F&_ga=2.24174973.678858453.1738585961-544272390.1734613155&_gac=1.89569769.1738174164.Cj0KCQiAwOe8BhCCARIsAGKeD565C8l-mK1yclAL_kGba0GKbG4nG2RjaRTQU6NTbAqbeZ7uzOyF0jQaArhGEALw_wcB&_gl=1*1o0anzw*_gcl_aw*R0NMLjE3MzgxNzQxNjQuQ2owS0NRaUF3T2U4QmhDQ0FSSXNBR0tlRDU2NUM4bC1tSzF5Y2xBTF9rR2JhMEdLYkc0bkcyUmphUlRRVTZOVGJBcWJlWjd1ek95RjBqUWFBcmhHRUFMd193Y0I.*_gcl_au*MTk3NjQ5NTQ3My4xNzM0NjEzMTU0*_ga*NTQ0MjcyMzkwLjE3MzQ2MTMxNTU.*_ga_D2D3EGKSGD*MTczODU4NTk2MS42LjEuMTczODU4NjIzNC42MC4wLjA.) is an event streaming platform used to collect, process, store, and integrate data at scale. It has numerous use cases including distributed logging, stream processing, data integration, and pub/sub messaging.

In order to make complete sense of what Kafka does, we'll delve into what an "event streaming platform" is and how it works. So before delving into Kafka architecture or its core components, let's discuss what an event is. This will help explain how Kafka stores events, how to get events in and out of the system, and how to analyze event streams.

## What Are Events?
An event is any type of action, incident, or change that's identified or recorded by software or applications. For example, a payment, a website click, or a temperature reading, along with a description of what happened.

In other words, an event is a combination of notification—the element of when-ness that can be used to trigger some other activity—and state. That state is usually fairly small, say less than a megabyte or so, and is normally represented in some structured format, say in JSON or an object serialized with Apache Avro™ or Protocol Buffers.

## Kafka and Events – Key/Value Pairs
Kafka is based on the abstraction of a distributed commit log. By splitting a log into partitions, Kafka is able to scale-out systems. As such, Kafka models events as key/value pairs. Internally, keys and values are just sequences of bytes, but externally in your programming language of choice, they are often structured objects represented in your language’s type system. Kafka famously calls the translation between language types and internal bytes serialization and deserialization. The serialized format is usually JSON, JSON Schema, Avro, or Protobuf.

Values are typically the serialized representation of an application domain object or some form of raw message input, like the output of a sensor.

Keys can also be complex domain objects but are often primitive types like strings or integers. The key part of a Kafka event is not necessarily a unique identifier for the event, like the primary key of a row in a relational database would be. It is more likely the identifier of some entity in the system, like a user, order, or a particular connected device.

This may not sound so significant now, but we’ll see later on that keys are crucial for how Kafka deals with things like parallelization and data locality.