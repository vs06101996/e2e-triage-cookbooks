---
id: demo-aws-sn-discovery-recover-cvo
jira: NOC-DEMO
defect_type: Demo / Sample
test_aliases:
  - Discovering CVO basic scenario flow
  - AWS single node - Discovery tests
failure_signatures:
  - awssndisc6605
  - Received: undefined
  - recoverCvo
  - awsSN.discovery.test.ts
  - Discovering CVO basic scenario
source_files:
  - tests/e2e/aws-sn/awsSN.discovery.test.ts
---

## Identify

Discovery test fails on `recoverCvo`: expected CVO name `awssndisc6605` but received `undefined`.

## Debug

1. Open `awsSN.discovery.test.ts` around the `recoverCvo` assertion.
2. Check whether recover API returned a body without `data.name`.

## Validate

Evidence shows discovery/recover assertion with undefined name — matches this demo cookbook.

## Resolution

Demo only — not a production recurring issue. For triage agent POC video.
