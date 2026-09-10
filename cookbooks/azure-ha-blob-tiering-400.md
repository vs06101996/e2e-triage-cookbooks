---
id: azure-ha-blob-tiering-400
jira: NOC-88888
defect_type: Product Issue
test_aliases:
  - Source cvo is down -Node1 is stopped- Happy Path
  - Azure High availability - High availability functional tests
  - Source cvo is down -Node2 is stopped- Happy Path
  - {"diskType":"Premium_LRS","disk":{"size":500,"unit":"GB"}}
  - Azure HA node - capacity tests
failure_signatures:
  - AxiosError: Request failed with status code 400 at settle (node_modules/
  - AxiosError: Request failed with status code 500 at settle (node_modules/
  - handleStreamEnd
  - endReadableNT
source_files:
  - tests/commonOccmTests/capacityOccmTests.ts
  - src/operations/cvoOps.ts
  - tests/commonOccmTests/volumeOccmTests.ts
  - tests/commonOccmTests/replicationOccmTests.ts
  - src/utils/helpers/replicationHelper.ts
---

## Identify

Azure HA Blob/tiering APIs returned 400 while the sibling non-tiering aggregate path in the same suite did not fail: create-aggregate with capacityTier Blob, change-tier-level, and Fabric Pool volume create with capacityTier Blob. Separate 500s on HA node-stop happy-path replication come from createCvoReplication after only delay(30000), without waitForCvoStatus(DEGRADED).

Seen on Azure HA build [2558830](https://teamcity.platform.bluexp.netapp.com/viewLog.html?buildId=2558830&buildTypeId=Jetty_E2E_Azure_AzureHaOccmE2eNewInfra) — Tests failed: 6 (4 new), passed: 102.

`tests/e2e/azure-ha/azureHA.capacity.test.ts: ● Azure HA node - capacity tests › Aggregate creation for each disk type, add disks, delete › with tiering enabled testing: {"diskType":"Premium_LRS","disk":{"size":500,"unit":"GB"}} AxiosError: Request failed with status code 400 at settle (node_modules/axios/lib/core/settle.js:19:12) at IncomingMessage.handleStreamEnd (node_modules/axios/lib/adapte...`

## Debug

Azure HA Blob/tiering APIs returned 400 while the sibling non-tiering aggregate path in the same suite did not fail: create-aggregate with capacityTier Blob, change-tier-level, and Fabric Pool volume create with capacityTier Blob.

1. `tests/commonOccmTests/capacityOccmTests.ts` — testCvoDataDiskCapByInstanceType() (L721)
2. `src/operations/cvoOps.ts`
3. `tests/commonOccmTests/volumeOccmTests.ts`
4. `tests/commonOccmTests/replicationOccmTests.ts`
5. `src/utils/helpers/replicationHelper.ts`

Four new 400s isolate to Azure data-tiering, not generic infra. Functional 500s match a known weak wait after stopHighAvailabilityCvoInstance.

## Validate

Confidence from phase-3 code RCA: 0.68. Suspected mechanism: Azure HA Blob/tiering APIs returned 400 while the sibling non-tiering aggregate path in the same suite did not fail: create-aggregate with capacityTier Blob, change-tier-level, and Fabric Pool volume create with capacityTier Blob.

TODO(human): state what the fix looks like so a future match can confirm the fix is present.

## Resolution

- Jira: NOC-88888
- Fix: TODO(human) — PR / commit
