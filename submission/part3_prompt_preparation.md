# Part 3: Prompt Preparation

## Repository: aiokafka  
Repo Link: https://github.com/aio-libs/aiokafka  
PR Link: https://github.com/aio-libs/aiokafka/pull/193  

---

## 3.1.1 Repository Context

The `aiokafka` repository is a Python library used to connect Python applications with Apache Kafka. Kafka is a distributed system that is mainly used to send and receive messages between different services. This repository helps Python developers work with Kafka using asynchronous programming.

The main purpose of this repository is to make Kafka communication easier in Python applications which use `asyncio`. Many modern backend systems are event-driven, where different services exchange information in real time. In such systems, asynchronous processing is useful because it improves performance and avoids blocking operations.

This repository provides features like producing messages to Kafka topics and consuming messages from topics. It also supports consumer groups, partition handling, offset management, security, and message compression. These features are important in large systems where many applications depend on stream-based communication.

The intended users are backend developers, data engineers, and software teams working on distributed systems. Developers use this library when building applications such as log processing systems, real-time notifications, analytics systems, or communication between microservices.

The repository solves the problem of integrating Kafka with Python in an asynchronous way. Other Kafka clients may use threads or synchronous methods, but this library is designed specifically for Python async frameworks such as FastAPI and aiohttp. This makes it useful for modern scalable applications.

Overall, aiokafka is mainly used where reliable real-time communication is needed between different services running in distributed environments.

---

## 3.1.2 Pull Request Description

This pull request focuses on improving how the consumer behaves when Kafka returns an invalid offset. Offset means the position from where a consumer reads messages. Sometimes the stored offset becomes unavailable because Kafka removes old messages after retention cleanup.

Before this pull request, if a consumer tried to read from an old offset that was already deleted, the application could fail or stop consuming messages. This created issues in long-running services because manual restart was required.

The PR updates the consumer behavior so that it handles such cases properly. When an offset is not found, the system checks the `auto_offset_reset` configuration. Depending on the setting, it moves the consumer either to the earliest available offset or the latest offset.

This change is useful when services are down for some time and then restart. During that time, Kafka may remove older records. With this fix, the consumer automatically recovers without crashing.

The previous behavior was less reliable because invalid offsets caused failures. After this update, the consumer can recover automatically. The pull request also updates related test cases to check that reset behavior works correctly.

This improvement makes the consumer more stable and reduces manual effort for developers. It also matches expected Kafka behavior for handling offset errors.

---

## 3.1.3 Acceptance Criteria

✓ When offset becomes invalid, consumer should detect the error and apply reset.

✓ When `auto_offset_reset = earliest`, consumer should start from first available record.

✓ When `auto_offset_reset = latest`, consumer should start from latest available record.

✓ Consumer should continue working without stopping application.

✓ Only affected partition should reset, not all partitions.

✓ Existing consumer features should work normally.

✓ Tests should verify correct reset behavior.

---

## 3.1.4 Edge Cases

### Case 1
Consumer offset points to deleted messages because of Kafka retention.

### Case 2
One partition has invalid offset while other partitions are valid.

### Case 3
Partition rebalance happens during offset reset process.

### Case 4
Temporary broker error occurs while fetching offset.

---

## 3.1.5 Initial Prompt

You are working on the aiokafka repository, which is an asynchronous Kafka client library for Python. The repository is used for sending and receiving Kafka messages using asyncio.

Implement the changes from pull request #193. The main issue is related to invalid offsets during message consumption. Sometimes Kafka removes old messages because of retention, and the consumer stored offset may no longer exist.

Currently, when this happens, the consumer may fail and stop processing. This should be fixed so that the consumer automatically applies the configured offset reset policy.

Required changes:

- Detect out-of-range offset errors.
- Apply `auto_offset_reset` setting.
- If set to `earliest`, read from earliest offset.
- If set to `latest`, read from latest offset.
- Keep unaffected partitions running.
- Prevent consumer crash.
- Maintain stable partition assignment.

Acceptance criteria:

- Consumer should recover automatically.
- No manual restart required.
- Single partition and multiple partitions should work.
- Existing logic should not break.
- Tests should pass successfully.

Edge cases:

- Deleted messages due to retention.
- Rebalance during reset.
- Invalid offsets in multiple partitions.
- Broker connection issue.
- Metadata refresh problem.

Testing:

- Add unit tests.
- Add integration tests.
- Verify offset reset.
- Verify partition consistency.
- Check no regression in existing behavior.

Expected result:

The consumer should continue processing messages correctly even when stored offsets become invalid.