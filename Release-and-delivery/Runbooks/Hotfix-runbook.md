# Hotfix runbook

> **Draft.** Depends on [ADR006](../Decisions/ADR006-Handle-PRD-incidents-with-a-hotfix-process.md) (proposed). Hotfix branches need release tags and release candidates; until those exist, use route H1 and approve only the newest run.

## 1. Is it a hotfix?

| Severity        | Examples                                                                                                                              | Response                               |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| **Sev 1**       | AI Portal down or claims cannot be opened; data of the wrong claim shown; personal data exposed; actively exploitable vulnerability   | Hotfix now                             |
| **Sev 2**       | Core journey degraded for many users (summary or timeline not loading); event ingestion stopped; critical vulnerability with exposure | Hotfix within the business day         |
| **Sev 3 and 4** | Cosmetic issues, bugs with a workaround                                                                                               | Not a hotfix: normal fix, next release |

**The Engineering Team** declares the hotfix and its severity. The developer on duty does the work (see the [release captain rotation](./Release-to-PRD.md#release-captain-rotation)). **QA** tests the fix and runs the smoke test in ACC and PRD.

## 2. Roll back first?

If the last release went out **less than 48 hours ago** and going back would restore service: **roll back first** with the [Rollback runbook](./Rollback-runbook.md), then fix calmly.

## 3. Tell the team

Post in the team channel:

> 🔥 **Hotfix started**, Sev [1 or 2]: [what users see]. Cause: [known or under investigation]. Who: [developer on duty], decision by [The Engineering Team]. Next update at [time].

If users notice the problem, the PO informs them.

## 4. Choose the route

::: mermaid
graph TD
A["Hotfix declared"] --> B{"Only configuration? secret, env var, CPU or memory"}
B -->|"yes"| H4["H4: emergency branch in k8s-deployments"]
B -->|"no"| C{"Is main releasable? no unfinished or risky work since the last release"}
C -->|"yes"| H1["H1: urgent PR on main"]
C -->|"no"| H2["H2: hotfix branch from the release tag"]
:::

### H1: urgent PR on main (default)

1. Create a branch from `main`, fix, add a regression test.
2. Open a PR with title `fix(#<ticket>): <what>` and tag the work item `hotfix`.
3. One reviewer who is not the author approves. The PR pipeline must be green.
4. Merge. CI creates a new release candidate; CD deploys it to TST.
5. Promote to ACC, QA runs the smoke test, promote to PRD (PO approves). Never skip ACC.

### H2: hotfix branch from the release tag

Use it when `main` contains work that must not go to PRD yet.

1. Create the branch from the release tag of what PRD runs (normally the newest; after a rollback, the tag of the release PRD went back to):
   - `git fetch --tags`
   - `git switch -c hotfix/<ticket> release/<date>`
2. Make the smallest possible fix and add a regression test. Squash it into **one commit** before you push: a CI run on a new branch compares only with the previous commit (`HEAD~1`), so apps changed in earlier commits would not be rebuilt.
3. Post in the team channel which files the hotfix touches, and ask the team not to merge changes to those files until the hotfix is merged back.
4. Open a PR from `hotfix/<ticket>` to `main` for the review (complete it later). The PR pipeline checks the hotfix merged with the current `main`, so conflicts show up now, before PRD.
5. Run CI on `hotfix/<ticket>`. It creates a release candidate from the hotfix branch.
6. Promote that release candidate to ACC, QA runs the smoke test, then PRD (PO approves).
7. After PRD the pipeline tags `release/<date>.1`.
8. **The same day, after PRD is confirmed OK:** complete the PR so the fix is in `main` and in the next release candidate. Use a normal merge (no squash), so the tagged commit becomes part of the history of `main` and the next changelog does not list the fix again.

**What happens to the work merged after the release tag?** Nothing. The hotfix branch only contains the release tag plus the fix. When it is merged back, git compares both sides from the release tag, so only the fix is added on top of `main`. The changes merged after the tag stay as they are, and they still do not reach PRD until the next release.

**If the PR to `main` has conflicts** (work merged after the tag changed the same lines):

1. Do not change the hotfix branch: its commit runs in PRD and carries the tag.
2. Create `hotfix/<ticket>-to-main` from `main`, cherry pick the fix (`git cherry-pick <hotfix commit>`), resolve the conflict and open a normal PR. Abandon the original PR.
3. If `main` rewrote that code, the fix in `main` may look different from the one in PRD. That is fine as long as the regression test passes in both.
4. Mention in the next release notes that this fix is already in PRD, because the changelog will list it again.

### H4: configuration only

**Secrets** come from AWS Secrets Manager, synced from Key Vault:

1. Change the value in Key Vault `kv-bic` ([ACC & PRD: Create & Update Secrets](https://dev.azure.com/NonLife/Zakelijk%20Schade/_wiki/wikis/Zakelijk-Schade.wiki/28595/ACC-&-PRD-Create-&-Update-Secrets)).
2. Run the PRD secret sync pipeline ([Sync Pipelines](https://dev.azure.com/NonLife/Zakelijk%20Schade/_wiki/wikis/Zakelijk-Schade.wiki/28597/Sync-Pipelines-Synchronize-Secrets-to-AWS)).
3. The pods are updated automatically after the sync. Check in ArgoCD that the app restarted and is Synced and Healthy.

**Other settings** live in `apps/nn-si-bic-claim-assistant/<app>/prd/values.json`: non-secret environment variables (`envVars`) and CPU and memory (`reqCPU`, `limCPU`, `reqMemory`, `limMemory`):

1. In k8s-deployments, create a branch starting with `emergency/`, for example `emergency/441999-bff-memory`.
2. Change only `envVars` or the CPU and memory fields.
3. Open and complete the PR. The `emergency/` prefix skips the Cloud Platform Team review; they get a Teams notification and review afterwards.
4. Make the same change for TST and ACC, so the environments stay aligned.

Do not change `image.tag` or `commit` by hand. To go back to a previous version, use the [Rollback runbook](./Rollback-runbook.md).

For the Felet Lambda (Cloud Team account) involve the Cloud team before deploying.

## 5. Check PRD

ArgoCD apps Synced and Healthy, `/health-check` and `/ready-check` respond, the QA smoke test passes, Splunk error rate back to normal for 15 minutes.

## 6. Close

Post in the team channel:

> ✅ **Hotfix done**: [what was fixed], in PRD since [time] as `release/<date>.1`. Postmortem on [date].

- [ ] For Sev 1 and Sev 2: a postmortem within 5 working days ([template](../Templates/Postmortem-template.md)).
- [ ] A work item for the root cause.
- [ ] Update this runbook if something was unclear.

## Related pages

- [ADR006 Handle PRD incidents with a hotfix process](../Decisions/ADR006-Handle-PRD-incidents-with-a-hotfix-process.md)
- [US 441563 Hotfix strategy and process](../Spikes/US-441563-Hotfix-strategy-and-process.md)
