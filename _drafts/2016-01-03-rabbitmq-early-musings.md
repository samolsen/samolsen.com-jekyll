---
layout: post
title:  "Early musings on RabbitMQ"
categories: software
---

Roughly six weeks ago at my [day job][castle-rock] we rolled a major
infrastructure change into production, replacing a JMS publish-and-subscribe
system (backed by a [JBoss Messaging][jboss-messaging] message broker, a part of
JBoss AS 5) with [RabbitMQ][rabbitmq] AMQP.

With the early returns in, I'm cautiously optimistic we have a messaging
solution which will meet our needs for the next few years. Here are some of my
thoughts about using RabbitMQ, and its [Java client library][rabbitmq-java] in
production.

#### The problem

We have hub-and-spoke system where writes all pass through a single node which,
after persisting the data, broadcasts deltas to N number of other nodes. Writes
come from a number of custom polling applications, or from a human-operated
content management system. This is not a big data firehose scenario. Messages
have larger payloads than many examples I've seen (3-130 KB), but are published
relatively infrequently.

A change in message broker was motivated by durable subscription problems with
the JBoss Messaging broker (JMB) in the face of network failures.

When using durable subscriptions, each client registers a unique id with the
broker (in certain implementations this may be a queue name). In the event of a
broken connection (absent client), the broker persists messages for the client
id so they may be delivered upon re-connecting.

The JMB would not recognize a broken connection, then refuse to allow a client
to reconnect due to its client id already already existing. The only mitigation
for this problem was our operations team changing the id for a client following
a network error, or performing some manual step to remove its queue so the
client id could be reused.

Replacing JMB became a priority. While there are projects like
[HornetQ][hornetq] and [ActiveMQ][activemq] that implement the JMS
specification, I felt it was worth investigating other message brokers
as long as subscriber clients would be updated anyway.

#### A tangent on serialization

In addition to changing the message broker, we also updated our system's
serialization mechanism. Previously, data was sent over the wire in XML format.
XML documents were build through a homemade factory class wrapping
`java.lang.StringBuilder`. This routine was slow (speed-wise on par with native
Java serialization), created a lot of heap garbage, and yielded large data
payload.

As a replacement, we're now using a number of [Jackson][jackson]
serializers/deserializers resulting in 7x faster serialization and a 60%
reduction in data payload.

Jackson is pretty great.

[castle-rock]: http://crc-corp.com
[rabbitmq]: https://www.rabbitmq.com
[rabbitmq-java]: https://www.rabbitmq.com/java-client.html
[jboss-messaging]: http://jbossmessaging.jboss.org
[jackson]: http://wiki.fasterxml.com/JacksonHome
[hornetq]: http://hornetq.jboss.org
[activemq]: http://activemq.apache.org
