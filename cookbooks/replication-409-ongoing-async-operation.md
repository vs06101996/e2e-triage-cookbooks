---
id: replication-409-ongoing-async-operation
jira: NOC-44195
defect_type: Automation Issue
test_aliases:
  - OnGoingAsyncOperationException
  - Delete replication and validate
failure_signatures:
  - OnGoingAsyncOperationException
  - Error: 409
  - ongoing operations which might interfere
  - Delete Replication
  - removeReplicationRelationship
  - breakReplicationRelationship
  - resyncReplicationRelationship
source_files:
  - replication operations layer
---

## Identify

HTTP 409 with message containing `OnGoingAsyncOperationException` and "Couldn't perform action Delete Replication, because there are ongoing operations which might interfere". Often hits delete/resync/edit-schedule tests on shared/pooled CVOs across Azure/GCP/AWS.

## Debug

1. Trace `removeReplicationRelationship` / `breakReplicationRelationship` / `resyncReplicationRelationship` — no retry on 409.
2. Multiple tests/suites can have baseline transfers or cleanup in flight on the same working environment.
3. OCCM error text is a "try again" contract — safe to retry only on this specific 409+exception, not all errors.

## Validate

Confirm `withConflictRetry()` wrapper: detects 409 + `OnGoingAsyncOperationException`, retries with backoff (default 10 × 30s) on mutating calls only.

## Resolution

- Jira: NOC-44195
- Fix: PR #683 (commit 726cb422)
- Cross-linked to NOC-44194 lockstep-retry regression
