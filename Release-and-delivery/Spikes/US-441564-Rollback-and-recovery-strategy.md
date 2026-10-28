# US 441564 Rollback and recovery strategy

|          |                                                                                                                                                  |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Spike    | [US 441564: 4. Spike - Rollback and Recovery Strategy](https://dev.azure.com/NonLife/Synera%202.0/_workitems/edit/441564)                        |
| Epic     | [439240 Techdebt 2026](https://dev.azure.com/NonLife/Synera%202.0/_workitems/edit/439240)                                                        |
| Status   | Draft for the alignment meeting                                                                                                                  |
| Decision | [ADR007 Roll back by redeploying the previous release candidate](../Decisions/ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md) |
| Runbook  | [Rollback runbook](../Runbooks/Rollback-runbook.md)                                                                                              |
| Related  | US 388710 (rollback in ArgoCD), US 390288 (DLQ handling)                                                                                         |

## The question

How do we bring PRD back to the previous release quickly, safely and visibly, for every part we deploy?

## Summary

There is no rollback process today, and the ArgoCD rollback button does not work with our GitOps setup. We propose: **redeploy the previous release candidate through our own pipeline** (minutes, nothing rebuilt), with a rebuild from the release tag until release candidates exist and when artifacts expired. Rollback stays a manual decision, taken within 15 minutes. We keep the last 5 PRD releases and practise a rollback in ACC every quarter.

## Findings

- k8s-deployments states: "there is no automatic rollback mechanism". Options today: redeploy through CI, change `values.json` by hand (Cloud Platform Team review for ACC and PRD), or `git revert`.
- The `emergency/` prefix skips the Cloud Platform Team review out of hours.
- The ArgoCD rollback button does not stick: ArgoCD reapplies what Git says. That answers US 388710: roll back through Git.
- CDK and WSO2 rollback means redeploying older pipeline artifacts. How long we keep runs and images is not defined.
- Rebuilding from a release tag is incomplete today: a CI run on a tag finds no earlier successful run on that ref and uses `HEAD~1` as the NX base, so only the apps changed in the last commit are rebuilt. Quick win N5 fixes this.
- Secrets (AWS Secrets Manager, synced from Key Vault) and the settings in `values.json` (`envVars`, CPU and memory) are not part of the release. A rollback only changes the image, not them.
- Our apps have no database. Problems in downstream systems (AI Brain, FELET) need a fix forward.

## Feasibility per part

| Part                      | State                                      | Rollback                                                         | Watch out                                                                  |
| ------------------------- | ------------------------------------------ | ---------------------------------------------------------------- | -------------------------------------------------------------------------- |
| AI Portal                 | No state                                   | ✓ easy                                                           | Roll back together with the BFF if the shared contract changed             |
| BFF                       | No state, no database                      | ✓ easy                                                           | Contract with AI Brain                                                     |
| Document retrieval API    | No state                                   | ✓ easy                                                           | none                                                                       |
| Event ingestion API       | No state, asynchronous                     | ✓ image; ⚠ events processed by the bad version may need a replay | DLQ handling (US 390288)                                                   |
| Event ingestion CDK stack | AWS resources and Lambda                   | ⚠ redeploy the previous synthesis                                | Replaced or deleted resources cannot come back: review the CDK diff in PRs |
| Felet Lambda stack        | Cloud Team account                         | ⚠ cross-account redeploy                                         | Coordinate with the Cloud team                                             |
| WSO2 configuration        | Gateway definitions                        | ⚠ republish the previous configuration                           | Check API revision behaviour                                               |
| Secrets                   | AWS Secrets Manager, synced from Key Vault | Not part of a release                                            | Revert in Key Vault and run the secret sync pipeline                       |
| `envVars`, CPU and memory | `values.json` in k8s-deployments           | Not part of a release                                            | Revert with a PR in k8s-deployments                                        |
| Data                      | No database                                | Roll forward                                                     | Downstream issues need a fix forward                                       |

## Options

| Option                                            | Speed           | Covers           | Notes                                                          |
| ------------------------------------------------- | --------------- | ---------------- | -------------------------------------------------------------- |
| R1 Feature flag off                               | seconds         | flagged features | Separate topic (US 441556)                                     |
| **R2 Redeploy the previous release candidate** ⭐ | minutes         | apps, CDK, WSO2  | Our pipeline, no wait; needs kept artifacts                    |
| R3 Rebuild from the release tag                   | tens of minutes | everything       | Until RCs exist and when artifacts expired; needs quick win N5 |
| R4 ArgoCD rollback button                         |                 |                  | ✗ does not stick, bypasses the audit trail                     |
| R5 Automatic canary or blue-green                 | automatic       | Kubernetes apps  | Future; needs Argo Rollouts from the Cloud Platform Team       |

We do not change `values.json` in k8s-deployments by hand: a rebuild from the tag covers everything and goes through our pipeline.

Full comparison: [ADR007](../Decisions/ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md).

## The proposed process

- **Triggers:** Sev 1 within 48 hours after a release; error rate above the agreed threshold (for example 5xx above 2% for 10 minutes in Splunk); failing health, ready or smoke checks; a security exposure.
- **Decision:** release captain or The Engineering Team, within 15 minutes; the PO is informed.
- **Mechanisms in order:** R2, then R3.
- **Validation after a rollback:** ArgoCD apps Synced and Healthy; `/health-check` and `/ready-check` respond; smoke test passes; Splunk error rate back to normal for 15 minutes.
- **Retention:** keep at least the last 5 PRD release candidates: pipeline runs with a retention lease, images, CDK and WSO2 artifacts. Release candidates that reached ACC get a 30 day retention lease.
- **Rehearsal:** a rollback drill in ACC every quarter.
- **Communication and follow-up:** a message in the team channel, an incident work item, root cause, a plan to release again.
- **Metrics (DORA):** deployment frequency, lead time from merge to PRD, change failure rate, time to restore. Baseline: since April the Portal went to PRD 16 times, the BFF 15, document retrieval 8 and event ingestion 8.

## Example: a bug in the BFF causes 500 errors

1. Alert, Sev 1, rollback decided.
2. The release captain starts the CD pipeline for the previous release candidate with `ROLLBACK=true`; the guard allows the older version.
3. Only the BFF differs (and the Portal if the contract changed), so only those are updated. The k8s-deployments PRs complete within a minute and ArgoCD syncs.
4. Validation passes, team channel message; PRD matches the previous release tag again.
5. The fix follows the [Hotfix runbook](../Runbooks/Hotfix-runbook.md).

## Acceptance criteria coverage

| Acceptance criterion                          | Where                                                                                                                                                                         |
| --------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Investigation document                        | This page                                                                                                                                                                     |
| Feasibility matrix per project                | Feasibility per part                                                                                                                                                          |
| Artifact strategy (versions, tags, retention) | Retention, [ADR004](../Decisions/ADR004-Tag-PRD-releases-and-generate-the-changelog.md), [Release candidates explained](../How-releases-work/Release-candidates-explained.md) |
| Trigger criteria                              | Triggers                                                                                                                                                                      |
| Decision workflow                             | Decision, [ADR006](../Decisions/ADR006-Handle-PRD-incidents-with-a-hotfix-process.md)                                                                                         |
| Database rollback strategy                    | No database; roll forward downstream                                                                                                                                          |
| Mechanism options                             | Options, [ADR007](../Decisions/ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md)                                                                             |
| Integration with release and hotfix           | ADR003, ADR006                                                                                                                                                                |
| Role of feature flags                         | R1, separate topic                                                                                                                                                            |
| Health checks and validation                  | Validation after a rollback                                                                                                                                                   |
| Rollback testing                              | Rehearsal, [Rollback runbook](../Runbooks/Rollback-runbook.md)                                                                                                                |
| NX dependency analysis                        | Feasibility per part (coupled pairs)                                                                                                                                          |
| Retention policy                              | Retention (to confirm)                                                                                                                                                        |
| Communication and post-rollback actions       | Communication and follow-up                                                                                                                                                   |
| Example failure scenario                      | Example                                                                                                                                                                       |
| Success metrics                               | Metrics                                                                                                                                                                       |
| Recommendation and team alignment             | ADR007, alignment meeting                                                                                                                                                     |

## Open questions

- Do image lifecycle rules in Artifactory and ECR keep at least 5 PRD versions?
- What error rate threshold do we agree for a rollback?
- WSO2: does republishing an older configuration create a new API revision without side effects?
