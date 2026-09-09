---
id: onprem-replication-stale-state-race
jira: NOC-44196
defect_type: Automation Issue
test_aliases:
  - ReSync replication and validation
  - Delete replication and validation
failure_signatures:
  - mirrorState
  - snapmirrored
  - uninitialized
  - unhealthyReason
  - onPremReplicationOccmTests.ts
  - toMatchObject
  - Cannot read properties of undefined
source_files:
  - tests/commonOccmTests/onPremReplicationOccmTests.ts
  - src/utils/utils.ts
---

## Identify

Two symptoms on the same code area: (1) `toMatchObject` mismatch on `mirrorState` — expected `snapmirrored`, received `uninitialized`. (2) `TypeError: Cannot read properties of undefined (reading 'unhealthyReason')` on delete. Both in `onPremReplicationOccmTests.ts`.

## Debug

1. Resync: test captures `getReplicationRelationshipDetails()` once after creation, breaks+resyncs, then asserts against stale pre-break snapshot before baseline reached `snapmirrored`.
2. Delete: single fetch immediately post-delete with unconditional `.unhealthyReason` access; relationship may already be fully removed.
3. Root cause: point-in-time reads racing the async SnapMirror state machine.

## Validate

Confirm `retryUntil()` poller in `utils.ts`: resync polls for `mirrorState === 'snapmirrored'` before and after resync; delete polls until destroyed or gone, with `if (rs)` guard before field access.

## Resolution

- Jira: NOC-44196
- Fix: PR #684 (commit 279ee342)
