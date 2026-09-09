---
id: replication-wrong-relationship-index
jira: NOC-44194
defect_type: Automation Issue
test_aliases:
  - Create replication and validation
  - Delete replication and validation
failure_signatures:
  - VsaWorkingEnvironment-
  - OnPremWorkingEnvironment-
  - relationships[0]
  - replicationHelper.ts
  - replicationOccmTests.ts
  - expect(received).toEqual(expected)
  - was not found
source_files:
  - src/utils/helpers/replicationHelper.ts
  - tests/commonOccmTests/replicationOccmTests.ts
---

## Identify

Look for `expect(received).toEqual(expected)` where Expected uses `VsaWorkingEnvironment-*` but Received uses `OnPremWorkingEnvironment-*` (or the reverse). Also appears as delete-source-volume errors: expected snapmirror relationship message but received "Volume ... was not found". Stack frames point to `replicationHelper.ts` and `replicationOccmTests.ts`.

## Debug

1. Open `validateReplicationCreation` / `validateReplicationCreationStatus` in `replicationHelper.ts` and confirm assertion uses `relationships[0]` from `getReplicationStatus()`.
2. Check suite topology: VSA↔VSA and VSA↔OnPrem replication suites share a pooled connector and run as parallel Jest workers.
3. Root cause: `[0]` is whichever relationship the backend returned first, not necessarily the one this test created — intermittent cross-suite pollution.

## Validate

Confirm fix is present: `findReplicationStatusForCvos()` filters by `workingEnvironmentId` and volume names instead of blind `[0]` indexing. `createCvoReplication()` returns exact volume/svm names for deterministic filtering.

## Resolution

- Jira: NOC-44194
- Fix: PR #685 (commit 2b4d1903)
- Follow-on: type fix 4ba89d94; serialized deletes 257017ff (cross-linked NOC-44195)
