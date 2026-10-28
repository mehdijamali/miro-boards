# ADR003 - Promote release candidates to PRD

- **Status**: proposed
- **Deciders**: BIC team (Engineers, PO)
- **Date**: 2026-10-19
- **Scope**: team
- **PR**: none yet; reviewed on the wiki and in the alignment meeting

## Context and Problem Statement

This is a team level decision about how we release our apps (AI Portal, BFF, document retrieval API, event ingestion API), our CDK stacks and our WSO2 configuration to PRD.

Today a PRD release is **whatever one pipeline run built**: the apps NX marked as affected in that run. Between mid-June and 15 October 2026, 23 of 39 runs from `main` timed out at the ACC approval, so their changes only reached PRD if a later run happened to rebuild the same apps.

Approving an older run after a newer one silently deploys older images. And `k8s-deployments` records the wrong commit, so nobody can say what runs in PRD without digging through pipeline logs. The full evidence is in [US 391474 PRD release orchestration](../Spikes/US-391474-PRD-release-orchestration.md).

**Question:** what is a release, and how do we make sure it contains everything we merged, never goes back by accident, and can be explained afterwards?

## Decision Drivers

- **No missed changes:** everything merged to `main` reaches PRD with the next release.
- **No accidental rollback:** an old approval can never deploy an older version without saying so.
- **Know what is in PRD:** for every environment we can answer "**_which version, built from which commit, released when_**".
- **Fast rollback:** going back to the previous release takes minutes, without rebuilding.
- **Build once:** the image tested in ACC is the image that goes to PRD.
- **Low daily overhead and fits our GitOps setup** (k8s-deployments and ArgoCD, owned by the Cloud Platform Team).

## Considered Options

- A. **Keep the current process**: approve the right runs, re-run what was missed
- B. **Since the last release tag**: tag every PRD release and use that tag as the NX base
- C. **Release candidates**: every merge creates a manifest with the version of every app; promote it
- D. **Release branches**: cut a release branch per release
- E. **Always build and deploy everything**
- F. **A separate pipeline and approvals per app**

## Pros and Cons of the Options

### A. Keep the current process

- Good, because it needs no work.
- Bad, because changes are still missed and old approvals can still roll PRD back.
- Bad, because it depends on people remembering which runs to approve, and on dummy commits.

### B. Since the last release tag

After every PRD release the pipeline tags the commit. CI uses `nx affected --base=<latest release tag>`, so every run builds everything changed since the last release. The Score team adopted this after hitting the same problem.

- Good, because it is a small change and proven in SI.
- Good, because nothing merged is forgotten, and tags give history.
- Bad, because every run rebuilds a growing set of apps until the next release.
- Bad, because approving an older run can still deploy older images.
- Bad, because a rollback means rebuilding from an old tag.

### C. Release candidates

Every CI run on `main` publishes a release candidate: the version of **every** app plus the CDK and WSO2 artifacts. CD compares the RC with the environment and only updates what differs. A guard refuses an older RC unless the run is a deliberate rollback. See [Release candidates explained](../How-releases-work/Release-candidates-explained.md).

- Good, because nothing merged can be forgotten: an RC is the whole system.
- Good, because the guard blocks accidental rollbacks.
- Good, because the image tested in ACC is the image in PRD.
- Good, because rollback is "promote the previous RC", without rebuilding.
- Good, because CI still builds only what changed and only changed apps restart.
- Good, because it fits GitOps: k8s-deployments already holds the state per environment.
- Bad, because it needs more pipeline work (manifest, comparison, guard).
- Bad, because we need a retention policy for images and pipeline runs.
- Bad, because it is a new concept for the team.

### D. Release branches

- Good, because the content of a release is explicit and frozen.
- Bad, because of branch management overhead and merge back mistakes.
- Bad, because it conflicts with trunk based development and NX affected on `main`.

### E. Always build and deploy everything

- Good, because it is simple and nothing can be missed.
- Bad, because every release restarts every app, and images are rebuilt at release time, so PRD gets bits that ACC did not test.

### F. A pipeline per app

- Good, because one app can be released or rolled back alone.
- Bad, because 5 apps times 3 environments means 15 approval steps.
- Bad, because coupled pairs (Portal with BFF, event ingestion API with its CDK stack) need manual coordination.
- Bad, because it has the highest maintenance.

### Comparison

|                           | A    | B   | C   | D    | E   | F    |
| ------------------------- | ---- | --- | --- | ---- | --- | ---- |
| No missed changes         | ✗    | ✓   | ✓   | ✓    | ✓   | ~    |
| No accidental rollback    | ✗    | ~   | ✓   | ✓    | ~   | ~    |
| Know what is in PRD       | ✗    | ✓   | ✓   | ✓    | ✓   | ~    |
| Fast rollback             | ✗    | ~   | ✓   | ~    | ~   | ✓    |
| Same image in ACC and PRD | ~    | ~   | ✓   | ~    | ✗   | ✓    |
| Only changed apps restart | ~    | ~   | ✓   | ✓    | ✗   | ✓    |
| Effort                    | none | S   | M   | M    | S   | L    |
| Daily overhead            | high | low | low | high | low | high |

## Decision Outcome

Proposed option: **"C. Release candidates"**, because it is the only option that meets all five decision drivers. If the team wants the smallest first step, B is acceptable as an interim: its release tag step is reused by C.

Release rhythm, proposed as a team agreement (not part of this ADR): release when it is valuable, at least once per sprint. See [Release to PRD](../Runbooks/Release-to-PRD.md).

Release candidates are **cumulative**: each one contains everything merged before it, so we cannot release merge 1 and 3 without merge 2. The rule that comes with this decision: **only merge to `main` what may go to PRD with the next release**. Unfinished or blocked work stays in its PR (and can be shown in the preview environment); work that is already merged and must not go out is reverted. Feature flags (US 441556) can help later.

## Positive Consequences

- Every release contains everything merged before it, and the pipeline shows what it contains before approval.
- The ACC approval timeout stops being a problem: the newest RC always contains everything.
- Rollback becomes a normal, fast pipeline action ([ADR007](./ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md)).
- Hotfixes use the same mechanism ([ADR006](./ADR006-Handle-PRD-incidents-with-a-hotfix-process.md)).

## Negative Consequences

- Pipeline work: RC manifest in CI, comparison and guard in CD, all CDK stacks and WSO2 configuration per RC. See [Set up release candidates](../Runbooks/Set-up-release-candidates.md).
- Follow-up checks: republishing unchanged WSO2 configuration must be harmless; `cdk deploy --all` must stay fast enough, including the cross-account Felet Lambda.
- We must keep the last 5 PRD RCs (runs, images, artifacts).
- `main` must stay releasable: we cannot leave one merged change out of a release.
- Dependency updates keep rebuilding every app until the Nx 23 upgrade gate of ADR002 is met. RCs handle that correctly.

## Prepared by

- BIC Claim Assistant team, from spike US #391474

## Related ADR

- [ADR004 Tag PRD releases and generate the changelog](./ADR004-Tag-PRD-releases-and-generate-the-changelog.md)
- [ADR005 Test feature branches in a preview environment](./ADR005-Test-feature-branches-in-a-preview-environment.md)
- [ADR006 Handle PRD incidents with a hotfix process](./ADR006-Handle-PRD-incidents-with-a-hotfix-process.md)
- [ADR007 Roll back by redeploying the previous release candidate](./ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md)
- ADR002 Manage dependency versions with a pnpm catalog (repository, PR 100291)
