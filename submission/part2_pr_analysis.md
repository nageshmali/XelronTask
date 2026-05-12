# PR Analysis – aiokafka

**Repo:** [aio-libs/aiokafka](https://github.com/aio-libs/aiokafka)

---

## PRs I picked

- PR #193 – gave the coordinator its own socket + added seek APIs
- PR #232 – fixed a crash in GroupCoordinator when a commit fails during rejoin

---

## PR #193

**Link:** https://github.com/aio-libs/aiokafka/pull/193

### Summary

So the problem here was that the GroupCoordinator (the part that manages things like heartbeats and offset commits) was sharing the same connection as all the fetch/produce traffic. Because of this, when the connection was busy with a big batch of messages, offset commits had to wait in line. That caused slow commits which is a known issue in this project (issues #137 and #128).

This PR fixes that by giving the coordinator its own separate socket. Now commit and heartbeat messages go through one connection, and the data traffic goes through another. On top of that, two new methods were also added – `seek_to_beginning()` and `seek_to_end()` – so users can jump to the start or end of a partition without doing it manually.

### Technical Changes

- `aiokafka/consumer/group_coordinator.py` – main file changed. it now opens its own connection instead of sharing one
- `aiokafka/consumer/consumer.py` – two new methods added here: `seek_to_beginning()` and `seek_to_end()`
- `aiokafka/consumer/fetcher.py` – small update to support the new seek methods
- `tests/test_consumer.py` – new tests for the seek methods and to check commits are not getting delayed
- `CHANGES.rst` – changelog updated under v0.3.0

### Implementation Approach

Before this, there was one client connection doing everything. This PR makes the coordinator open its own connection to the broker. So when heartbeats or commits need to go out, they don't have to wait for fetch requests to finish first.

For `seek_to_beginning` and `seek_to_end`, they internally call `fetcher.request_offset_reset()` with either EARLIEST or LATEST. The fetcher then sends a ListOffsets request to the broker and updates where the consumer is reading from. From the user side it's just one async function call, nothing complicated.

### Potential Impact

Any consumer using a group_id will get faster commits after this change. The two seek methods are useful when you need to replay messages from the beginning or skip ahead to latest. Before this PR, slow commits during heavy load was a real problem in production so this fix matters quite a bit.

---

## PR #232

**Link:** https://github.com/aio-libs/aiokafka/pull/232

### Summary

This PR fixes a bug where the consumer would fully stop working if an offset commit failed at the wrong moment. The specific place is inside `_on_join_prepare()` – a function that runs just before the consumer tries to rejoin a consumer group. If the commit inside that function threw an error (say the broker was having issues), the error was not caught anywhere and it would crash the whole coordination loop. Once that crashed, the consumer would never rejoin the group and you'd have to manually restart it.

The fix is pretty simple – just wrap the commit call in a try/except so if it fails, it logs a warning and keeps going with the rejoin anyway. There's also a second small fix in the same PR: `session_timeout_ms` was not being passed correctly to the GroupCoordinator, so whatever timeout the user set was being ignored.

### Technical Changes

- `aiokafka/consumer/group_coordinator.py` – two changes here: added try/except around `commit_offsets()` inside `_on_join_prepare()`, and fixed the `session_timeout_ms` not being passed correctly to the constructor
- `tests/test_consumer.py` – test added that makes a commit fail during `_on_join_prepare` to confirm the consumer still rejoins fine
- `CHANGES.rst` – two changelog entries added under v0.4.x

### Implementation Approach

The issue was that `_on_join_prepare()` was calling `await self.commit_offsets()` with no try/except around it. If the broker returned an error (timeout, rebalance already in progress, etc.), this would throw an exception. That exception then went up the call stack through `_do_rejoin_group` and into `_coordination_routine` which killed the background coordination task completely.

The fix wraps that commit call in a try/except. If it fails, it just logs a warning and moves on to the rejoin. The rejoin still happens normally. This is the same way the official Java Kafka client handles this situation – the commit before a rejoin is considered best effort, not a hard requirement.

For `session_timeout_ms`, the value was passed with the wrong keyword name so the coordinator ignored it and used the broker default of 10 seconds. One line fix but it means user configured timeouts now actually work.

### Potential Impact

This affects every consumer that's part of a consumer group. Before this fix, one bad commit during a rebalance could silently stop the consumer permanently. It's the kind of bug that's hard to reproduce because it only shows up under specific timing. After this fix the consumer handles it and keeps running. The timeout fix also matters for anyone who had custom session timeout values – those will now be used correctly.

---