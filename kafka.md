# Kafka: a Distributed Messaging System for Log Processing

## Introduction

Log data includes:
- user activity events like logins, pageviews, and clicks
- operational metrics like service call stack, call latency, errors, and system metrics like CPU utilization

Real-time usage of log data in production is hard because of massive volume.

Early systems would physically scrape log files off of servers.

More recent methods involve distributed log aggregators like Flume.

Such log aggregators are designed to load logs into a data warehouse for offline consumption.

Kafka combines the benefits of log aggregators and messaging systems:
- distributed, scalable, high throughput
- messaging system-like API
- allows consumption of events in real-time

Kafka allows for both online and offline consumption of log data

## Kafka Architecture and Design Principles

A *topic* defines a stream of messages of a particular type.

A producer can publish messages to a topic.

The published messages are stored in *broker* servers.

A consumer can subscribe to topics and pull messages from brokers.

A consumer can create multiple streams for one topic that will have an even distribution of messages.

Kafka exposes an iterator inferface for consumers.

Kafka supports:
- point-to-point delivery: multiple consumers consume a single copy of the stream
- publish-subscribe: multiple consumers consume their own copies of the stream

A kafka cluster consists of multiple brokers. Topics are divided into multiple partitions.

### Efficiency on a Single Partition

Each topic partition corresponds to a logical log. 

Physically, a log is a set of segment files. Producing a message appends a message to the last segment file.



