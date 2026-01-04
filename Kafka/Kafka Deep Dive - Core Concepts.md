
## Table of Contents

1. [Offsets and Consumer Tracking](#offsets-and-consumer-tracking)
2. [Consumers vs Partitions](#consumers-vs-partitions)
3. [Consumer Rebalancing](#consumer-rebalancing)
4. [Cooperative Rebalancing](#cooperative-rebalancing)
5. [Backpressure](#backpressure)

---

## Offsets and Consumer Tracking

### What are Offsets?

**Offsets are sequential position numbers (bookmarks) for messages in a partition.**

```
Partition 0: [msg0, msg1, msg2, msg3, msg4, msg5, ...]
Offsets:      0     1     2     3     4     5
```

### Key Points

- **Each consumer GROUP** (not individual consumer) maintains its own set of offsets
- One offset per partition that the group is consuming from
- Offsets are stored in a special Kafka topic: `__consumer_offsets`
- Multiple consumer groups can read the same topic with different offsets

**Example:**

```python
# Conceptual offset storage:
{
    "consumer-group-1": {
        "topic-x-partition-0": 1500,
        "topic-x-partition-1": 2300,
        "topic-x-partition-2": 890
    },
    "consumer-group-2": {
        "topic-x-partition-0": 45,    # Different group, different offsets
        "topic-x-partition-1": 120,
        "topic-x-partition-2": 67
    }
}
```

### Offset Tracking in Consumer Groups

- Only ONE consumer in a group is assigned to each partition at any time
- No conflict: each consumer commits offsets only for its assigned partitions
- Offsets can be committed automatically or manually

**Auto-commit Example:**

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'my-topic',
    bootstrap_servers=['localhost:9092'],
    group_id='my-consumer-group',
    enable_auto_commit=True,
    auto_commit_interval_ms=5000,  # Every 5 seconds
)
```

**Manual Commit Example:**

```python
consumer = KafkaConsumer(
    'my-topic',
    bootstrap_servers=['localhost:9092'],
    group_id='my-consumer-group',
    enable_auto_commit=False,
)

for message in consumer:
    process(message)
    consumer.commit()  # Explicit offset commit
```

### Configuring Starting Position

The `auto_offset_reset` parameter controls where to start consuming:

```python
# Start from EARLIEST message (beginning)
consumer = KafkaConsumer(
    'my-topic',
    bootstrap_servers=['localhost:9092'],
    group_id='new-group',
    auto_offset_reset='earliest'  # Start from offset 0
)

# Start from LATEST message (only new messages)
consumer = KafkaConsumer(
    'my-topic',
    bootstrap_servers=['localhost:9092'],
    group_id='new-group',
    auto_offset_reset='latest'  # Start from current end
)

# Throw error if no offset exists
consumer = KafkaConsumer(
    'my-topic',
    auto_offset_reset='none'
)
```

**Important:** `auto_offset_reset` only applies when:

1. The consumer group is NEW (no stored offset exists), OR
2. The stored offset is invalid/out of range

If a group already has a committed offset, it resumes from there.

---

## Consumers vs Partitions

### The Golden Rule

**One partition can only be consumed by ONE consumer in a consumer group at a time.**

But one consumer can handle multiple partitions.

### More Consumers than Partitions

```
Topic with 3 partitions: [P0] [P1] [P2]
Consumer Group with 5 consumers: C1, C2, C3, C4, C5

Assignment:
C1 → P0
C2 → P1
C3 → P2
C4 → IDLE ❌
C5 → IDLE ❌
```

**Result:** Extra consumers stay idle and waste resources.

**Code Example:**

```python
consumer = KafkaConsumer(
    'my-topic',  # Has 3 partitions
    bootstrap_servers=['localhost:9092'],
    group_id='same-group',
    consumer_timeout_ms=1000
)

print(f"Assigned partitions: {consumer.assignment()}")
# 3 instances will have partitions
# 2 instances will print: set() - they're idle
```

### Fewer Consumers than Partitions

```
Topic with 5 partitions: [P0] [P1] [P2] [P3] [P4]
Consumer Group with 2 consumers: C1, C2

Assignment:
C1 → P0, P1, P2
C2 → P3, P4
```

**Result:** Each consumer handles multiple partitions. No partitions are idle.

### Who Decides the Numbers?

**Partitions:** Decided when creating the topic:

```bash
kafka-topics.sh --create --topic my-topic \
    --partitions 10 \
    --replication-factor 3 \
    --bootstrap-server localhost:9092
```

**Partition Count Considerations:**

- More partitions = higher parallelism
- More partitions = more overhead (metadata, file handles)
- Rule of thumb: Based on throughput needs

**Consumers:** Decided when deploying your application:

```bash
# Deploy 3 instances
docker-compose scale my-consumer=3

# Or in Kubernetes:
kubectl scale deployment my-consumer --replicas=3
```

**Best Practice:** `num_consumers ≤ num_partitions` (ideally equal)

---

## Consumer Rebalancing

### What is Rebalancing?

When consumer group membership changes (join/leave/crash), Kafka redistributes partition assignments among remaining/new consumers.

### Why Rebalancing is Complex

Kafka needs to ensure:

1. No duplicate processing during handoff
2. No partition left unassigned
3. Even distribution of load
4. Coordination across all consumers

### Traditional (Eager) Rebalancing Process

**Step-by-step:**

1. **Trigger detected** - Consumer crash, join, or heartbeat timeout
2. **ALL consumers STOP consuming** - Even healthy ones! (Stop-the-world)
3. **Consumers revoke ALL partitions**
4. **Group Coordinator computes new assignment**
5. **New assignments sent to all consumers**
6. **Consumers resume with new partitions**

**Example Scenario:**

```
Initial state:
Consumer1 → [P0, P1, P2]
Consumer2 → [P3, P4, P5]
Consumer3 → [P6, P7, P8]

Consumer2 crashes:

1. Kafka detects failure
2. REBALANCE TRIGGERED
3. Consumer1 and Consumer3 STOP (even though healthy!)
4. All partitions released
5. New assignment:
   Consumer1 → [P0, P1, P2, P3, P4]
   Consumer3 → [P5, P6, P7, P8]
6. Both resume
```

### Rebalance Listeners

```python
from kafka import KafkaConsumer, ConsumerRebalanceListener

class MyRebalanceListener(ConsumerRebalanceListener):
    def on_partitions_revoked(self, revoked):
        print(f"Partitions revoked: {revoked}")
        # Cleanup, commit offsets, etc.
        
    def on_partitions_assigned(self, assigned):
        print(f"Partitions assigned: {assigned}")
        # Initialize state for new partitions

consumer = KafkaConsumer(
    'my-topic',
    bootstrap_servers=['localhost:9092'],
    group_id='my-group'
)

consumer.subscribe(['my-topic'], listener=MyRebalanceListener())
```

### Problem with Eager Rebalancing

- **Unavailability window:** ALL consumers stop during rebalance
- If 1 out of 100 consumers dies, the other 99 unnecessarily stop
- Can take seconds, causing processing delays

---

## Cooperative (Incremental) Rebalancing

### The Key Difference

|Eager|Cooperative|
|---|---|
|Everyone stop! Reorganize everything!|Only affected consumers stop. Others keep working.|

### How Cooperative Rebalancing Works

**Multi-phase process:**

```
Initial state:
Consumer1 → [P0, P1, P2]
Consumer2 → [P3, P4, P5]  ← CRASHES
Consumer3 → [P6, P7, P8]

Phase 1 (First rebalance):
- Consumer1 continues: P0, P1, P2 ✓
- Consumer3 continues: P6, P7, P8 ✓
- Coordinator identifies P3, P4, P5 orphaned
- Coordinator: "Consumer1, prepare for P3"

Phase 2 (Incremental):
- Consumer1 still processes P0, P1, P2 ✓
- Consumer1 picks up P3
- Consumer3 still processes P6, P7, P8 ✓
- Coordinator: "Consumer3, prepare for P4"

Phase 3:
- Consumer3 picks up P4
- Coordinator: "Consumer1, take P5"

Final state:
Consumer1 → [P0, P1, P2, P3, P5]
Consumer3 → [P6, P7, P8, P4]
```

### Configuration

```python
from kafka import KafkaConsumer
from kafka.coordinator.assignors.cooperative_sticky import CooperativeStickyAssignor

consumer = KafkaConsumer(
    'my-topic',
    bootstrap_servers=['localhost:9092'],
    group_id='my-group',
    partition_assignment_strategy=[CooperativeStickyAssignor]
)
```

### Rebalance Listener with Cooperative Mode

```python
class MyRebalanceListener(ConsumerRebalanceListener):
    def on_partitions_revoked(self, revoked):
        # Only called for partitions being moved AWAY
        print(f"Giving up partitions: {revoked}")
        
    def on_partitions_assigned(self, assigned):
        # Called for NEW partitions being assigned
        print(f"Taking new partitions: {assigned}")
        
    def on_partitions_lost(self, lost):
        # Called when partitions lost unexpectedly
        print(f"Lost partitions: {lost}")
```

### Benefits

**Comparison:**

- **Eager:** 100 consumers, 1 fails → All 100 stop (~5-10 seconds unavailability)
- **Cooperative:** 100 consumers, 1 fails → Only 1 partition pauses (~500ms), 99 keep running

---

## Backpressure

### What is Backpressure?

**Backpressure occurs when downstream components (consumers) can't keep up with upstream flow (producers).**

**Analogy:** Highway traffic

- Cars (messages) flow onto highway (Kafka)
- Exit ramps (consumers) remove cars
- If exits are slow, cars pile up
- Eventually highway becomes congested

### Why Does Backpressure Happen?

**1. Consumer Processing Slower than Production**

```python
# Producer: Fast (1000 msg/sec)
producer = KafkaProducer(bootstrap_servers=['localhost:9092'])
for i in range(1000):
    producer.send('my-topic', f'message-{i}'.encode())

# Consumer: Slow (10 msg/sec)
consumer = KafkaConsumer('my-topic', ...)
for message in consumer:
    process_heavy_task(message)  # Takes 100ms per message
    # Can only process 10/sec but receiving 1000/sec!
```

**2. Insufficient Consumer Instances**

```
Producer rate: 10,000 msg/sec
Partitions: 10
Consumers: 2
Each consumer handles: 5 partitions = 5,000 msg/sec
If each can only handle 1,000 msg/sec → Backpressure!
```

**3. Slow External Dependencies**

```python
for message in consumer:
    # Slow external API call
    response = requests.post('http://slow-api.com', data=message.value)
    # API takes 500ms, messages arrive every 1ms
    # Queue grows!
```

### Detecting Backpressure

**Monitor Consumer Lag:**

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'my-topic',
    group_id='my-group',
    enable_auto_commit=False
)

# Check lag
for partition in consumer.assignment():
    committed = consumer.committed(partition)  # Where consumer is
    end_offset = consumer.end_offsets([partition])[partition]  # Latest
    lag = end_offset - committed
    print(f"Partition {partition.partition}: Lag = {lag}")
    # Growing lag = backpressure!
```

### Solutions to Backpressure

**1. Scale Consumers Horizontally**

```bash
kubectl scale deployment my-consumer --replicas=10
```

**2. Increase Partitions**

```bash
kafka-topics.sh --alter --topic my-topic --partitions 20
```

**3. Optimize Consumer Processing**

```python
# Bad: Sequential
for message in consumer:
    slow_operation(message)  # Blocks!

# Better: Batch processing
messages = consumer.poll(timeout_ms=1000, max_records=500)
for topic_partition, records in messages.items():
    batch_process(records)  # Process 500 at once

# Even better: Async
import asyncio

async def process_message(msg):
    await async_slow_operation(msg)

for message in consumer:
    asyncio.create_task(process_message(message))
    # Don't wait! Move to next immediately
```

**4. Apply Backpressure Controls**

```python
# Limit fetch size
consumer = KafkaConsumer(
    'my-topic',
    max_poll_records=100,  # Fetch only 100 at a time
    max_partition_fetch_bytes=1048576,  # Max 1MB per partition
    fetch_max_wait_ms=500
)

# Pause/Resume consumption
from kafka import TopicPartition

for message in consumer:
    process(message)
    
    if is_overwhelmed():
        consumer.pause(*consumer.assignment())  # Pause!
        time.sleep(5)  # Catch up
        consumer.resume(*consumer.assignment())
```

**5. Increase Retention**

```bash
# Give more time to catch up
kafka-configs.sh --alter --topic my-topic \
    --add-config retention.ms=604800000  # 7 days
```

### Monitoring Example

```python
import time

def monitor_lag(consumer):
    """Monitor and alert on backpressure"""
    partitions = consumer.assignment()
    
    for partition in partitions:
        committed = consumer.committed(partition)
        if committed is None:
            continue
            
        end_offset = consumer.end_offsets([partition])[partition]
        lag = end_offset - committed
        
        if lag > 10000:
            print(f"⚠️  HIGH LAG on {partition}: {lag} messages!")
        elif lag > 1000:
            print(f"⚠️  Moderate lag on {partition}: {lag}")

# In consumer loop
while True:
    records = consumer.poll(timeout_ms=1000)
    
    for topic_partition, messages in records.items():
        for message in messages:
            process(message)
        consumer.commit()
    
    if int(time.time()) % 10 == 0:
        monitor_lag(consumer)
```

---

## Summary Table

|Concept|Key Insight|
|---|---|
|**Offsets**|Per consumer-group, per partition bookmark. Stored in `__consumer_offsets` topic|
|**Consumer:Partition Ratio**|1 partition = 1 consumer max (per group). Extra consumers idle. One consumer can handle multiple partitions|
|**Eager Rebalancing**|Stop-the-world: ALL consumers pause during rebalance|
|**Cooperative Rebalancing**|Incremental: Only affected partitions pause, others keep working|
|**Backpressure**|Consumers can't keep up with producers. Monitor lag, scale consumers, optimize processing|

---

## Quick Reference Commands

### Create Topic

```bash
kafka-topics.sh --create --topic my-topic \
    --partitions 10 \
    --replication-factor 3 \
    --bootstrap-server localhost:9092
```

### Alter Partitions

```bash
kafka-topics.sh --alter --topic my-topic --partitions 20
```

### Check Consumer Groups

```bash
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
    --group my-group --describe
```

### Change Retention

```bash
kafka-configs.sh --alter --topic my-topic \
    --add-config retention.ms=604800000
```

---

## Best Practices

1. **Partitions:** Set based on throughput requirements
2. **Consumers:** Keep `num_consumers ≤ num_partitions`
3. **Rebalancing:** Use cooperative rebalancing for production
4. **Commits:** Manual commits for critical data, auto-commit for non-critical
5. **Monitoring:** Always monitor consumer lag
6. **Scaling:** Scale horizontally (more consumers) before vertically
7. **Testing:** Test rebalancing behavior under failure scenarios

---

_End of Notes_