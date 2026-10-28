# ADR006 Handle PRD incidents with a hotfix process

- Status: proposed
- Deciders: BIC team (Engineers, PO)
- Date: 2026-10-19
- Scope: team
- PR: none yet; reviewed on the wiki and in the alignment meeting

## Context and Problem Statement

This is a team level decision about how we fix a serious problem in PRD that cannot wait for the next release.

Today we have no agreed hotfix criteria, branch strategy, approvals or communication. The research is in [US 441563 Hotfix strategy and process](../Spikes/US-441563-Hotfix-strategy-and-process.md).

**Question:** when is something a hotfix, and how do we get a fix to PRD quickly without skipping safety checks or losing the fix later?

## Decision Drivers

- Restore service fast for serious problems.
- Keep the same safety: review, tests, ACC before PRD.
- Never lose a fix: it must end up in `main` and in the next release.
- One process the whole team knows, with clear roles.
- Works with release candidates ([ADR003](./ADR003-Promote-release-candidates-to-PRD.md)) and rollback ([ADR007](./ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md)).

## Considered Options

- H1. Fix forward on `main`: an urgent PR, a new release candidate, promoted quickly
- H2. A hotfix branch from the last release tag, merged back to `main`
- H3. Cherry pick into a release branch
- H4. Configuration only: Key Vault for secrets, an `emergency/` branch in k8s-deployments for the other settings in `values.json` (`envVars`, CPU and memory)

## Pros and Cons of the Options

### H1. Fix forward on main

- Good, because it uses the normal path, so there is no merge back.
- Bad, because it also releases everything else already merged to `main`, so `main` must be releasable.

### H2. Hotfix branch from the release tag

`hotfix/<ticket>` from the newest `release/*` tag, built into a release candidate, deployed to ACC and PRD, tagged `release/<date>.1`, and merged back to `main` the same day. The Score team uses this.

- Good, because only the fix goes to PRD. Work merged to `main` after the release tag is not affected: merging back only adds the fix.
- Bad, because it needs merge back discipline, and CI and CD must accept `hotfix/*` branches.
- Bad, because a conflict with work merged after the release tag must be resolved when merging back.

### H3. Cherry pick into a release branch

- Bad, because it only works with release branches, which we do not use ([ADR003](./ADR003-Promote-release-candidates-to-PRD.md) option D).

### H4. Configuration only

- Good, because it is the fastest: no build. The `emergency/` prefix skips the Cloud Platform Team review; they review afterwards.
- Bad, because it can only change configuration (secrets through Key Vault; `envVars`, CPU and memory in `values.json` through an `emergency/` branch), not code.

## Decision Outcome

Proposed: **H1 by default, H2 when `main` is not releasable, H4 for configuration only.**

**When is it a hotfix?**

| Severity    | Examples                                                                                                                              | Response                                                  |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| Sev 1       | AI Portal down or claims cannot be opened; data of the wrong claim shown; personal data exposed; actively exploitable vulnerability   | Hotfix now; roll back first if the last release caused it |
| Sev 2       | Core journey degraded for many users (summary or timeline not loading); event ingestion stopped; critical vulnerability with exposure | Hotfix within the business day                            |
| Sev 3 and 4 | Cosmetic issues, bugs with a workaround                                                                                               | Normal fix in the next release                            |

**Rules:**

- If the last release (less than 48 hours ago) caused it and a rollback restores service: roll back first ([ADR007](./ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md)), then fix calmly.
- Never skip ACC; it takes minutes. At least one reviewer who is not the author.
- Approvals: The Engineering Team declares the hotfix and severity; the PO approves PRD.
- Communication in the team channel at start and end. A postmortem within 5 working days for Sev 1 and Sev 2.
- H2 merge back: open the PR to `main` at the start, so the PR pipeline shows conflicts before PRD. Complete it the same day, after PRD is confirmed, with a normal merge (no squash) so the tagged commit is in the history of `main`. On a conflict, do not change the hotfix branch: cherry pick the fix onto a branch from `main` and resolve it there.
- No PRD release while a hotfix PR to `main` is still open, so a release never undoes a fix.

Steps: [Hotfix runbook](../Runbooks/Hotfix-runbook.md). Support outside office hours is not defined today; that is a management decision, outside this ADR.

## Positive Consequences

- Everyone knows when something is a hotfix and what to do.
- A hotfix never blocks the next release: it is tagged `.1` and merged back, so the next release candidate contains it.

## Negative Consequences

- H2 needs CI and CD to accept `hotfix/*` branches and branch control to allow them.
- H2 needs the branch policy on `main` to allow a normal merge (no squash) for `hotfix/*` PRs.
- H2 needs CI to use the release tag as the NX base for `hotfix/*` branches. Until then, a hotfix must be one commit, because a new branch falls back to `HEAD~1`.
- `main` must stay releasable for H1; unfinished work must not break PRD.

## Prepared by

- BIC Claim Assistant team, from spike US 441563

## Related ADR

- [ADR003 Promote release candidates to PRD](./ADR003-Promote-release-candidates-to-PRD.md)
- [ADR004 Tag PRD releases and generate the changelog](./ADR004-Tag-PRD-releases-and-generate-the-changelog.md)
- [ADR007 Roll back by redeploying the previous release candidate](./ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md)
