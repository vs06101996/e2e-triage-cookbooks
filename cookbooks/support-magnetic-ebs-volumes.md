---
id: support-magnetic-ebs-volumes
jira: NOC-99999
defect_type: Product Issue
test_aliases:
  - Support Magnetic EBS volumes
  - Azure Single node - Azure Working Environment tests
failure_signatures:
  - AzureSN.WorkingEnvironment.test.ts
  - expect(received).toEqual(expected)
  - testSupportingMagneticEbsVolume
  - workingEnvironmentOccmTests.ts
  - processTicksAndRejections
  - nssAccountId
  - toBeNull
source_files:
  - tests/commonOccmTests/workingEnvironmentOccmTests.ts
  - tests/e2e/azure-sn/AzureSN.WorkingEnvironment.test.ts
  - src/models/cvo.ts
  - src/utils/testConst.ts
---

## Identify

Support Magnetic EBS volumes failed on an NSS-id assertion, not disk type: CVO2 is created with nssAccount azureNssAccount (8669b95c-...), then testSupportingMagneticEbsVolume does a single getCvoFields(SupportRegistrationInformation) and expects nssAccountId to equal that UUID. The API returned null. Sibling fieldsProperties coverage polls this field via retryUntilLoaded because registration is eventually consistent; this WE test does not.

Seen on Azure SN build [2558832](https://teamcity.platform.bluexp.netapp.com/viewLog.html?buildId=2558832&buildTypeId=Jetty_E2E_Azure_AzureSnOccmE2eNewInfra) — Tests failed: 1, passed: 144, ignored: 6; error message: Process failed with code 1.

`tests/e2e/azure-sn/AzureSN.WorkingEnvironment.test.ts: ● Azure Single node - Azure Working Environment tests › Azure Working Environment tests- General tests › Support Magnetic EBS volumes expect(received).toEqual(expected) // deep equality Expected: "8669b95c-15e1-4bc4-8add-8e3bbceee828" Received: null 223 | expect(res[0].nssAccountId).toBeNull(); 224 | } else { > 225 | expect(res[0].nssAccoun...`

## Debug

Support Magnetic EBS volumes failed on an NSS-id assertion, not disk type: CVO2 is created with nssAccount azureNssAccount (8669b95c-...), then testSupportingMagneticEbsVolume does a single getCvoFields(SupportRegistrationInformation) and expects nssAccountId to equal that UUID.

1. `tests/commonOccmTests/workingEnvironmentOccmTests.ts` — workingEnvironmentOccmTests.ts (L1)
2. `tests/e2e/azure-sn/AzureSN.WorkingEnvironment.test.ts` — AzureSN.WorkingEnvironment.test.ts (L1)
3. `src/models/cvo.ts`
4. `src/utils/testConst.ts` — azureNssAccount (L59)

Stack and expected UUID pin the failure to workingEnvironmentOccmTests.ts:225 vs azureNssAccount. Create path passes nssAccount; assertion is one-shot unlike retryUntilLoaded for the same field.

## Validate

Confidence from phase-3 code RCA: 0.72. Suspected mechanism: Support Magnetic EBS volumes failed on an NSS-id assertion, not disk type: CVO2 is created with nssAccount azureNssAccount (8669b95c-...), then testSupportingMagneticEbsVolume does a single getCvoFields(SupportRegistrationInformation) and expects nssAccountId to equal that UUID.

TODO(human): state what the fix looks like so a future match can confirm the fix is present.

## Resolution

- Jira: NOC-99999
- Fix: TODO(human) — PR / commit
