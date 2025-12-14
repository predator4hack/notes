# Kafka

-   Super efficient MAIL ROOM

*   https://kafka.apache.org/intro/

-   https://learn.conduktor.io/kafka/what-is-apache-kafka/

System of moving data — especially streams of data

Think of it as a super-efficient mailroom that takes in messages from many sources, keeps track of them, and delivers them to the right destinations, often in real time.

## The Core Components

-   Producer: The sender—e.g., your project pushing data to a Kafka topic.
-   Kafka Topic: The mailbox—where data/messages arrive and wait for pickup.
-   Consumer: The receiver—applications or services that pick up the data/messages to do useful work.

EVENT DRIVEN PROCESSING - you trigger and event and don’t expect a response then and there

Points not covered in the above link:

## Kafka Offsets:

-   A Kafka offset is a unique identifier for each message (also called an event or record) within a specific partition of a Kafka topic
-   Ordered log, starts at 0 and increases by one for each new message sent to partition
-   Partition Specific, Monotonically increasing,

How does Kafka Chooses the Partition for the message:
Kafka producer/client application decide which partition that the message should be written to — using small program called partitioner

-   The key determines which partition a message goes into within a topic
-   Messages with the same key will always go to the same partition.
-   Within that partition, ordering of messages is guaranteed.

Intuitive Examples

-   Many updates per entity: For a user activity stream, use user_id so login, purchase, logout all stay together and ordered per user.
-   Single independent event: For system logs or monitoring events, use no key or a timestamp so events spread evenly and order is not required.
-   Grouping for batch operations: For order reprocessing, use order_id to ensure updates/processes for one order are never interleaved with those for others.

### Consumer Groups:

A Kafka consumer group is a collection of consumers (applications or services) that work together to read messages from one or more Kafka topics

When a group of consumers subscribes to a topic:

-   Each partition within that topic is assigned to only one consumer in the group.
-   No overlap: No two consumers in the group process the same partition simultaneously.
-   Parallelism: If you have more partitions than consumers, some consumers will process multiple partitions. If you have more consumers than partitions, some will remain idle.

### Partition Assignment Strategies

Kafka uses assignment strategies (configurable) to distribute partitions among consumers:

-   RangeAssignor: Divides partitions among consumers as evenly as possible.
-   RoundRobinAssignor: Rotates partition assignments for more balance.
-   StickyAssignor & CooperativeStickyAssignor: Balance assignments while minimizing changes when the group membership changes.

### Consumer Offsets:

-   Suppose a consumer reads message 2712 from partition 2 of the topic historical.reannotation.
-   It finishes its processing task (maybe updating a database or validation).
-   The consumer then issues a commit (automatically or manually), saying: "Offset 2712 for partition 2 is done."
-   Kafka stores this offset in its \_\_consumer_offsets topic.
-   If the consumer crashes and restarts, it resumes from offset 2713, picking up right where it left off.

### Delivery Semantic:

At Least Once Delivery (Preferred)

-   Meaning: Every event sent to Kafka will be delivered to the consumer at least one time, but sometimes it could be delivered more than once—resulting in duplicates.
-   How it works: The consumer reads a message, processes it (for example, updates a database), and only then commits the offset back to Kafka to mark it as processed. 1
-   If a crash happens before the commit: When the consumer restarts, Kafka will resend the same message, ensuring it was not missed—but it means the task might run twice. That’s why downstream logic should be idempotent (safe to run more than once).

At Most Once Delivery

-   Meaning: Each event is delivered zero or one time. Kafka will never deliver the same event twice, but there’s a risk some events might not be processed at all—especially in the event of a failure.
-   How it works: The consumer commits the offset as soon as the message is read (before processing). 1
-   If a crash happens during processing: The last-read message may be lost and never processed, since Kafka thinks the offset has already been handled.
