# Release candidates explained

> **Status: proposed** in [ADR003](../Decisions/ADR003-Promote-release-candidates-to-PRD.md). This page explains how releasing would work if the team accepts it. Nothing changes until then.

## In one minute

- Today a PRD release is **whatever one pipeline run happened to build**. If we approve the wrong run, or let a run time out, merged changes stay behind or PRD silently goes back to an older version.
- With release candidates, **every merge to `main` produces a release candidate (RC)**: a list with the version of **every** app and stack, not only the ones that changed.
- TST always runs the newest RC. Releasing means **promoting an RC** to ACC and then PRD. Only apps whose version differs are redeployed.
- Because the newest RC always contains everything, **nothing merged can be forgotten**. Because the pipeline knows the order of RCs, **an old approval cannot roll PRD back** by accident.
- Going back is a normal, visible action: **promote the previous RC**.

## Why we need it

Three things go wrong today. All three were seen between mid-June and 15 October 2026.

| Problem                 | What happens                                                                                                                                                                                         | Evidence                                                                                                                                            |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Missed changes**      | A pipeline run only carries the apps NX marked as affected for that run. If its ACC approval times out after 24 hours, those changes only reach PRD if a later run happens to rebuild the same apps. | 23 of 39 runs from `main` timed out at ACC. Automatic Summary 2.0 reached PRD on 30 September only because the next approved run rebuilt every app. |
| **Accidental rollback** | Approving an older run after a newer one deploys older images. Nothing warns you.                                                                                                                    | On 29 September two runs waited at ACC at the same time. Approving them in the wrong order would have removed a fix from PRD.                       |
| **Unknown PRD state**   | k8s-deployments records the NX base commit instead of the commit the image was built from, and there are no release tags.                                                                            | PRD says commit `af154d0`; it really runs an image built from `384f0878`.                                                                           |

Details and more evidence: [US 391474 PRD release orchestration](../Spikes/US-391474-PRD-release-orchestration.md).

## What a release candidate is

A release candidate is a small file (a manifest) that CI publishes for every successful run on `main`. It answers one question: **"if we release this commit, which version of each part goes to the environment?"**

| Part                                                        | What the RC holds                                         |
| ----------------------------------------------------------- | --------------------------------------------------------- |
| AI Portal, BFF, document retrieval API, event ingestion API | The image tag (build id) and the commit it was built from |
| Mockflow                                                    | The same, marked "never PRD"                              |
| Infra (CDK): event ingestion stack, Felet Lambda stack      | The pipeline run whose CDK output (`cdk.out`) to deploy   |
| WSO2 gateway configuration                                  | The pipeline run whose WSO2 artifacts to publish          |

Example (illustrative):

```json
{
  "rc": 559905,
  "commit": "4c86cff8",
  "apps": {
    "bic-claim-assistant-web": { "image": "559905", "builtFrom": "4c86cff8" },
    "bic-claim-assistant-api": { "image": "559905", "builtFrom": "4c86cff8" },
    "bic-document-retrieval-api": { "image": "549747", "builtFrom": "384f0878" },
    "bic-event-ingestion-api": { "image": "549747", "builtFrom": "384f0878" },
    "mockflow-api": { "image": "549747", "builtFrom": "384f0878", "prd": false }
  },
  "infra": { "event-ingestion-cdk": { "artifactsFromRun": 559905 }, "felet-lambda-cdk": { "artifactsFromRun": 559905 } },
  "wso2": { "artifactsFromRun": 559905 }
}
```

In this example only the Portal and the BFF changed in run 559905. The other apps keep the image of run 549747. CI still **builds only what changed**; the RC simply carries the other versions forward.

## The life of a release candidate

::: mermaid
graph LR
M1["Merge PR A"] --> RC1["RC 101"]
M2["Merge PR B"] --> RC2["RC 102"]
M3["Merge PR C"] --> RC3["RC 103"]
RC1 --> TST["TST runs the newest RC"]
RC2 --> TST
RC3 --> TST
RC3 -->|"release captain promotes"| ACC["ACC"]
ACC -->|"approval"| PRD["PRD"]
PRD --> TAG["Tag release/2026.10.20 and release notes"]
:::

1. **A PR is merged** into `main`. Its title follows Conventional Commits, for example `feat(#418043): show timeline annotations`.
2. **CI runs.** It builds the apps NX marks as affected since the **previous RC**, synthesizes all CDK stacks, prepares all WSO2 configuration and publishes the new RC.
3. **CD deploys the RC to TST automatically.** So TST always runs the newest RC, which is `main`.
4. **The release captain promotes an RC**, normally the newest. Before approving ACC, everyone sees the changelog: every PR since the last PRD release.
5. **ACC, then PRD.** In each environment the pipeline compares the RC with what the environment runs today (the `values.json` files in k8s-deployments) and only updates apps whose version differs. CDK deploys all stacks; CloudFormation skips stacks that did not change.
6. **After PRD**, the pipeline tags the commit `release/YYYY.MM.DD`, keeps the pipeline run, and posts a Teams message. The PO publishes the release notes.

## The guard: no accidental rollback

Before deploying, the pipeline checks whether the environment already runs a **newer** RC (with `git merge-base --is-ancestor`).

- Newer RC already there: the run **stops** with a clear message.
- Older or same RC: the run continues.
- To go back on purpose, start the run with `ROLLBACK=true`. That is a rollback, and it is visible in the pipeline history. See the [Rollback runbook](../Runbooks/Rollback-runbook.md).

## Example: three PRs, today and with release candidates

PR A changes the BFF, PR B changes the Portal, PR C changes the BFF again. Only the last run is approved.

|                              | Today                                                        | With release candidates                      |
| ---------------------------- | ------------------------------------------------------------ | -------------------------------------------- |
| What run 3 carries           | Only the BFF (A and C)                                       | RC 3: every app, including the Portal with B |
| What reaches PRD             | BFF with A and C. **PR B stays behind** and nobody is warned | A, B and C                                   |
| Someone approves run 1 later | PRD silently goes back to an older BFF                       | The guard stops the run                      |
| "What is in PRD?"            | Read pipeline logs                                           | Look at the release tag and its RC           |

## How hotfixes and rollbacks fit in

- **Hotfix**: normally a PR on `main`, which creates a new RC that we promote quickly. When `main` contains work that must not go to PRD yet, we branch `hotfix/<ticket>` from the last release tag; that branch also produces an RC. See [ADR006](../Decisions/ADR006-Handle-PRD-incidents-with-a-hotfix-process.md) and the [Hotfix runbook](../Runbooks/Hotfix-runbook.md).
- **Rollback**: promote the previous RC with `ROLLBACK=true`. Nothing is rebuilt, so it takes minutes. See [ADR007](../Decisions/ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md).

## What stays the same

- We still work trunk based on `main` with pull requests.
- CI still builds only the affected apps.
- k8s-deployments and ArgoCD still deploy our apps. The pipeline still creates the k8s-deployments PRs, which complete automatically.
- ACC and PRD still need approvals.

## What changes for you

| Role                    | Today                                                                                              | With release candidates                                                                                                                                                    |
| ----------------------- | -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Developers**          | Remember which runs to approve; dummy commits to force a deploy; run CI on branches to test in TST | Promote the newest RC; a `deployProjects` parameter instead of dummy commits; feature branches go to the [preview environment](../Runbooks/Use-the-preview-environment.md) |
| **PO and business**     | Unclear what a release contains; no release notes                                                  | See the changelog **before** approving; write short release notes; decide when to release                                                                                  |
| **UX/UI and front-end** | TST may run someone's branch                                                                       | TST always runs `main`; show work in progress in the preview environment                                                                                                   |

## Frequently asked questions

**Can we still release only the BFF?**
A release always promotes a complete RC, but only apps whose version differs from PRD are redeployed. If only the BFF changed since the last release, only the BFF restarts.

**What happens when a CI run fails?**
A failed run publishes no RC. The next successful run builds everything that changed since the last RC, so the failed changes are included automatically.

**What if more PRs are merged while ACC waits for approval?**
Nothing is lost. Approve the RC you tested, or promote the newer RC instead. The ACC timeout no longer matters, because the newest RC always contains everything. If the CD run of the RC you want expired, start the CD pipeline manually and pick that CI run under Resources.

**Can we release merge 1 and 3, but not 2?**
No, and that is on purpose. Each RC is the state of `main` after a merge, so RCs are cumulative: a combination of 1 and 3 never existed on `main` and was never tested. So we only merge what may go to PRD with the next release. Unfinished work stays in its PR (show it in the preview environment); merged work that must not go out is reverted.

**We released RC2 and rolled PRD back to RC0. Can we still release RC1?**
Yes, if change 1 does not depend on change 2 and change 2 is the cause of the problem.

1. ACC still runs RC2, which is newer than RC1, so start the CD run for RC1 with `ROLLBACK=true` for ACC. Smoke test RC1 there: it was never tested in ACC.
2. PRD runs RC0, which is older than RC1, so promote RC1 to PRD normally. The guard allows it, and only the apps that changed between RC0 and RC1 are redeployed.
3. The pipeline gives RC1 its first release tag. The tag of RC2 stays as history. The changelog starts from the commit PRD runs (RC0), so it lists change 1.

Keep in mind: RC1 never reached PRD, so it is kept by the 30 day retention lease from its ACC stage, not by the PRD retention. And `main` still contains change 2, so every new RC brings it back: fix or revert change 2 on `main` first. Often that is simpler anyway: the fix creates RC3 (change 1 plus the fix), which we promote through ACC to PRD as usual.

**Do CDK stacks and WSO2 change every release?**
The pipeline runs `cdk deploy` for all stacks and publishes all WSO2 configuration for every RC. Unchanged CDK stacks are skipped by CloudFormation. For WSO2 we still verify that republishing unchanged configuration is harmless.

**How many RCs do we keep?**
At least the last 5 PRD releases: pipeline runs, images and CDK and WSO2 artifacts. That is what makes a fast rollback possible.

**Is this an NN standard?**
There is no mandatory SI release process. Other teams use the same building blocks: the Score team uses the last release tag as the NX base, Compass shows the changelog per stage, and the Inkomen platform only deploys a version that already runs in the previous environment.

**What is the first step?**
The quick wins (correct NX base commit, CI never "partially succeeded", the real commit in k8s-deployments) are useful whatever we decide. Then we build the RC pipeline: see [Set up release candidates](../Runbooks/Set-up-release-candidates.md).

## Related pages

- [ADR003 Promote release candidates to PRD](../Decisions/ADR003-Promote-release-candidates-to-PRD.md): all options compared
- [Release tags and release notes](./Release-tags-and-release-notes.md)
- [Release to PRD](../Runbooks/Release-to-PRD.md): the release captain checklist
- [US 391474 PRD release orchestration](../Spikes/US-391474-PRD-release-orchestration.md): the full research
