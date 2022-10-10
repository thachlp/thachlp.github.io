---
layout: post
title: How to maintain Kafka client connectivity
tags: [kafka, kafka client]
comments: true
---
When Kafka is running, I mean producer/consumer connected to server and published/subscribed messages normally, what happen when Kafka server is down?
Today, to answer this, I try to stop Kafka server (on my local machine): 
- Firstly, the Kafka consumer is still running normally (maybe forever until the server is up), that what I expected.
- Secondly, the Kafka producer was terminated by these exceptions:
  - `Caused by: org.apache.kafka.common.errors.TimeoutException: Topic <> not present in metadata after 60000 ms.` <br/>
  The exception was thrown by this config parameter: `kafka.max.block.ms` is expired, the default is 60000s.
  - `Caused by: org.apache.kafka.common.errors.TimeoutException: Expiring 1 record(s) for <>:120010 ms has passed since batch creation`.<br/>
  The exception was thrown by this config parameter: `kafka.delivery.timeout.ms` is expired, the default is 120000s.

So to make the Kafka client keep running when server is down (wait for server up), we only need to increase the value of these configs of producer.
