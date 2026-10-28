# Rollback runbook

> **Draft.** Depends on [ADR007](../Decisions/ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md) (proposed). Route R2 needs release candidates; until they exist, use route R3 (rebuild from the release tag).

## 1. Decide (within 15 minutes)

Roll back when the last release causes one of these:

- A Sev 1 problem within 48 hours after the release.
- An error rate above the agreed threshold, for example 5xx above 2% for 10 minutes in Splunk.
- Failing `/health-check`, `/ready-check` or smoke tests.
- A security exposure.

Who decides: the **release captain or The Engineering Team**. Inform the PO.

Post in the team channel:

> ⏪ **Rollback started** in PRD: [what users see]. Going back from `release/<new>` to `release/<previous>`. Who: [name]. Next update at [time].

Do **not** use the rollback button in ArgoCD: ArgoCD reapplies what Git says, so it does not stick.

## 2. Route R2: redeploy the previous release candidate (default)

1. Find the previous release: the release tag before the current one, for example `release/2026.10.06`, and its pipeline run (the release candidate).
2. Start the CD pipeline for that release candidate with the parameter `ROLLBACK=true`. Select the environment to roll back (normally PRD only).
3. The guard sees an older release candidate and allows it because of `ROLLBACK=true`.
4. The pipeline only updates apps whose version differs. The k8s-deployments PRs complete within a minute and ArgoCD syncs. CDK and WSO2 are deployed from the artifacts of that release candidate.
5. Release tags stay as they are: the tag of the release you rolled back from stays as history. The next changelog starts from the commit PRD runs, so it stays correct.
6. Go to step 4, Check.

## 3. Route R3: rebuild from the release tag

Use it until release candidates exist, and later when the artifacts of the previous release are gone.

> **Prerequisite:** CI must rebuild every app that differs from the previous release. Today a CI run on a tag finds no earlier successful run on that ref and uses `HEAD~1` as the NX base, so it only rebuilds the apps changed in the last commit. Quick win N5 (`deployProjects` parameter) fixes this.

1. Find the previous release tag, for example `release/2026.10.06`.
2. Run the CI pipeline on that tag, with all apps.
3. The CD run deploys it to TST, then to ACC and PRD after approval.
4. Go to step 4, Check.

This takes tens of minutes and rebuilds the images. We do not change `values.json` in k8s-deployments by hand for a rollback.

**Secrets and `values.json` settings** are not part of a release: a rollback only changes the image. Secrets come from AWS Secrets Manager, synced from Key Vault; non-secret environment variables (`envVars`) and CPU and memory live in `values.json` in k8s-deployments. If the release came with a change to one of them, revert it separately: secrets in Key Vault and the secret sync pipeline ([ACC & PRD: Create & Update Secrets](https://dev.azure.com/NonLife/Zakelijk%20Schade/_wiki/wikis/Zakelijk-Schade.wiki/28595/ACC-&-PRD-Create-&-Update-Secrets), [Sync Pipelines](https://dev.azure.com/NonLife/Zakelijk%20Schade/_wiki/wikis/Zakelijk-Schade.wiki/28597/Sync-Pipelines-Synchronize-Secrets-to-AWS)), `values.json` settings with a PR in k8s-deployments.

## 4. Check

- [ ] ArgoCD apps Synced and Healthy.
- [ ] `/health-check` and `/ready-check` respond for every app that changed.
- [ ] QA smoke test: sign in to the AI Portal, open a claim, check summary, timeline and documents.
- [ ] Splunk error rate back to normal for 15 minutes.
- [ ] Event ingestion rolled back? Check the dead letter queue: events handled by the broken version may need a replay.

## 5. Close

Post in the team channel:

> ✅ **Rollback done**: PRD runs `release/<previous>` again since [time]. Fix plan: [hotfix or next release].

- [ ] Incident work item with the root cause.
- [ ] Plan the fix with the [Hotfix runbook](./Hotfix-runbook.md).
- [ ] For Sev 1 and Sev 2: a postmortem within 5 working days ([template](../Templates/Postmortem-template.md)).

## Per part

| Part                      | Rollback                             | Watch out                                            |
| ------------------------- | ------------------------------------ | ---------------------------------------------------- |
| AI Portal, BFF            | Easy                                 | Together if the shared contract changed              |
| Document retrieval API    | Easy                                 | none                                                 |
| Event ingestion API       | Easy for the image                   | Events may need a replay (DLQ)                       |
| Event ingestion CDK stack | Redeploy the previous synthesis      | Replaced or deleted resources do not come back       |
| Felet Lambda stack        | Cross-account redeploy               | Coordinate with the Cloud team                       |
| WSO2 configuration        | Republish the previous configuration | Check API revisions                                  |
| Secrets                   | Not part of a release                | Revert in Key Vault and run the secret sync pipeline |
| `envVars`, CPU and memory | Not part of a release                | Revert with a PR in k8s-deployments                  |

## Quarterly rollback drill (ACC)

Once per quarter, in ACC, during office hours:

1. Note the current release candidate in ACC.
2. Roll back ACC to the previous release candidate with route R2.
3. Run the checks of step 4 and time the whole rollback.
4. Promote the newest release candidate again.
5. Write down the duration and any problems in the drill log below, and update this runbook.

| Date | Who | From, to | Duration | Problems found |
| ---- | --- | -------- | -------- | -------------- |
|      |     |          |          |                |

## Related pages

- [ADR007 Roll back by redeploying the previous release candidate](../Decisions/ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md)
- [US 441564 Rollback and recovery strategy](../Spikes/US-441564-Rollback-and-recovery-strategy.md)
