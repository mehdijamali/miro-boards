# ADR007 Roll back by redeploying the previous release candidate

- Status: proposed
- Deciders: BIC team (Engineers, PO)
- Date: 2026-10-19
- Scope: team
- PR: none yet; reviewed on the wiki and in the alignment meeting

## Context and Problem Statement

This is a team level decision about how we bring PRD back to the previous release when a release breaks it.

k8s-deployments states that there is no automatic rollback mechanism. The options today are a redeploy through CI, a manual change of `values.json` (Cloud Platform Team review for ACC and PRD), or `git revert`. The rollback button in ArgoCD does not work for us: ArgoCD immediately applies what Git says again. For CDK and WSO2 a rollback means redeploying older pipeline artifacts, and how long we keep them is not defined. The research is in [US 441564 Rollback and recovery strategy](../Spikes/US-441564-Rollback-and-recovery-strategy.md).

**Question:** how do we go back to the previous release quickly, safely and visibly?

## Decision Drivers

- Restore service within minutes.
- No rebuild: deploy exactly what ran before.
- Visible and audited: a rollback shows up in the pipeline history and in Git.
- Covers apps, CDK stacks and WSO2 configuration.
- Works without waiting for another team, also outside office hours.

## Considered Options

- R1. Switch a feature off with a feature flag (separate topic, US 441556)
- R2. Redeploy the previous release candidate through our pipeline
- R3. Rebuild from the previous release tag
- R4. The rollback button in ArgoCD
- R5. Automatic canary or blue-green with Argo Rollouts

## Pros and Cons of the Options

### R1. Feature flag off

- Good, because it takes seconds and only affects one feature.
- Bad, because we have no feature flags yet. It is handled in a separate document and does not replace a rollback.

### R2. Redeploy the previous release candidate

A CD run for the previous RC with `ROLLBACK=true`. Apps go through the normal `update-deployment` pipeline, CDK and WSO2 from that RC's artifacts.

- Good, because it takes minutes and rebuilds nothing.
- Good, because pipeline created k8s-deployments PRs complete automatically, so there is no wait.
- Good, because it covers apps, CDK and WSO2, and it is visible in the history.
- Bad, because it needs release candidates ([ADR003](./ADR003-Promote-release-candidates-to-PRD.md)) and kept artifacts.

### R3. Rebuild from the release tag

Run CI on the previous release tag and deploy that run through TST, ACC and PRD.

- Good, because it covers everything (apps, CDK and WSO2) and works without release candidates or old artifacts (the Score team does this).
- Good, because it uses our normal pipeline, so it is visible in the history and needs no manual change in k8s-deployments.
- Bad, because it takes much longer (build, tests and three environments with approvals) and rebuilds the images, so PRD gets new bits.
- Bad, because today it does not rebuild every app: a CI run on a tag finds no earlier successful run on that ref and uses `HEAD~1` as the NX base, so only the apps changed in the last commit are built. Quick win N5 (`deployProjects` parameter) fixes this.

### R4. ArgoCD rollback button

- Bad, because ArgoCD immediately reapplies what Git says, so it does not stick, and it bypasses the audit trail. This also answers US 388710: roll back through Git.

### R5. Automatic canary or blue-green

- Good, because it can roll back automatically.
- Bad, because Argo Rollouts is not installed on the Cloud Platform Team clusters. It is a future option.

## Decision Outcome

Proposed: **R2 by default. R3 until release candidates exist, and later when the artifacts of the previous release are gone. R4 is not used, and we do not change k8s-deployments by hand for a rollback.**

- **Trigger:** a Sev 1 within 48 hours after a release, error rate above the agreed threshold (for example 5xx above 2% for 10 minutes in Splunk), failing health or smoke checks, or a security exposure.
- **Decision:** release captain or The Engineering Team, within 15 minutes; the PO is informed. Rollback stays a manual decision.
- **Retention:** keep at least the last 5 PRD release candidates: pipeline runs (retention lease), images and CDK and WSO2 artifacts. Release candidates that reached ACC get a 30 day retention lease, so one that never reached PRD can still be released after a rollback.
- **Rehearsal:** a rollback drill in ACC every quarter.

Steps: [Rollback runbook](../Runbooks/Rollback-runbook.md).

## Positive Consequences

- Rollback becomes a normal, practiced action that takes minutes.
- Every rollback is visible in the pipeline history and in k8s-deployments.

## Negative Consequences

- Needs release candidates first; until then R3 (rebuild from the release tag) is the route, and it needs quick win N5 first.
- Secrets and the settings in `values.json` (`envVars`, CPU and memory) are not part of a release: a rollback only changes the image. If a release came with a change to them, revert it separately: secrets in Key Vault and the secret sync pipeline, `values.json` settings with a PR in k8s-deployments.
- Image lifecycle rules in Artifactory or ECR must keep at least 5 PRD versions (to confirm).
- Events processed by a broken event ingestion version may need a replay (DLQ handling, US 390288).
- CDK resources that were replaced or deleted cannot be restored by a rollback; review the CDK diff in PRs.

## Prepared by

- BIC Claim Assistant team, from spike US 441564

## Related ADR

- [ADR003 Promote release candidates to PRD](./ADR003-Promote-release-candidates-to-PRD.md)
- [ADR006 Handle PRD incidents with a hotfix process](./ADR006-Handle-PRD-incidents-with-a-hotfix-process.md)
