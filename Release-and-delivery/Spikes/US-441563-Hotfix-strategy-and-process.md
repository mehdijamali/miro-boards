# US 441563 Hotfix strategy and process

|          |                                                                                                                        |
| -------- | ---------------------------------------------------------------------------------------------------------------------- |
| Spike    | [US 441563: 3. Spike - Hotfix Strategy and Process](https://dev.azure.com/NonLife/Synera%202.0/_workitems/edit/441563) |
| Epic     | [439240 Techdebt 2026](https://dev.azure.com/NonLife/Synera%202.0/_workitems/edit/439240)                              |
| Status   | Draft for the alignment meeting                                                                                        |
| Decision | [ADR006 Handle PRD incidents with a hotfix process](../Decisions/ADR006-Handle-PRD-incidents-with-a-hotfix-process.md) |
| Runbook  | [Hotfix runbook](../Runbooks/Hotfix-runbook.md)                                                                        |

## The question

When is a problem in PRD a hotfix, and how do we get a fix to PRD fast without skipping safety checks or losing the fix later?

## Summary

We have no hotfix process today. We propose: **fix forward on `main` by default**; a **hotfix branch from the last release tag** when `main` contains work that must not go to PRD yet; and a **configuration only** route for small fixes (Key Vault for secrets, an `emergency/` branch in k8s-deployments for `envVars`, CPU and memory). If the last release caused the problem, **roll back first**. A severity table decides what counts as a hotfix.

## Findings

- No documented hotfix criteria, branch strategy, approvals or communication.
- Any branch can reach ACC and PRD today; there is no branch control.
- The `emergency/` prefix in k8s-deployments skips the Cloud Platform Team review (they review afterwards). It only helps for changes in k8s-deployments, such as `envVars`, CPU or memory in `values.json`.
- Secrets come from AWS Secrets Manager, synced from Key Vault by the secret sync pipelines. Non-secret environment variables live in `envVars` in `values.json`.
- Our apps have no database, so a hotfix rarely needs data migration.
- Coupled pairs must move together: the Portal with the BFF, and the event ingestion API with its CDK stack.
- Support outside office hours is not defined.
- The Score team has a documented hotfix process: branch from the release tag, deploy, tag, merge back.

## Options

| Option                                                                     | How                                                                                          | Pros                                   | Cons                                                             |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | -------------------------------------- | ---------------------------------------------------------------- |
| **H1 Fix forward on `main`** ⭐ default                                    | Urgent PR, new release candidate, ACC, PRD                                                   | One path, no merge back                | Releases everything already merged, so `main` must be releasable |
| **H2 Hotfix branch from the release tag** ⭐ when `main` is not releasable | `hotfix/<ticket>` from `release/<date>`, release candidate, ACC, PRD, tag `.1`, merge back   | Only the fix goes out; proven by Score | Merge back discipline; CI must accept `hotfix/*`                 |
| H3 Cherry pick into a release branch                                       | Needs release branches                                                                       | Familiar                               | We do not use release branches                                   |
| H4 Configuration only                                                      | Key Vault and secret sync; `emergency/` branch in k8s-deployments for `envVars`, CPU, memory | Fastest, no build                      | Configuration only                                               |

Full comparison: [ADR006](../Decisions/ADR006-Handle-PRD-incidents-with-a-hotfix-process.md).

## The proposed process

**Severity**

| Severity    | Examples                                                                                                            | Response                                                  |
| ----------- | ------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| Sev 1       | AI Portal down or claims cannot be opened; wrong claim data shown; personal data exposed; exploitable vulnerability | Hotfix now; roll back first if the last release caused it |
| Sev 2       | Core journey degraded for many users; event ingestion stopped; critical vulnerability with exposure                 | Hotfix within the business day                            |
| Sev 3 and 4 | Cosmetic, workaround exists                                                                                         | Next release                                              |

**Decision flow**

::: mermaid
graph TD
A["Incident in PRD"] --> B{"Severity?"}
B -->|"Sev 3 or 4"| N["Normal fix, next release"]
B -->|"Sev 1 or 2"| C{"Last release less than 48 h ago and rollback helps?"}
C -->|"yes"| R["Roll back first, then fix"]
C -->|"no"| D{"Is main releasable?"}
D -->|"yes"| F["H1: urgent PR on main"]
D -->|"no"| H["H2: hotfix branch from the release tag"]
:::

**Roles** (R does, A decides, C consulted, I informed)

| Activity                                                  | Developer on duty | The Engineering Team | PO    | Release captain | QA    | Cloud Platform Team or Cloud team |
| --------------------------------------------------------- | ----------------- | -------------------- | ----- | --------------- | ----- | --------------------------------- |
| Declare hotfix and severity                               | R                 | **A**                | C     | I               | I     |                                   |
| Build and review the fix                                  | R                 | **A**                | I     | I               | C     |                                   |
| Test the fix and smoke test in ACC and PRD                | C                 | I                    | I     | I               | **R** |                                   |
| Approve PRD                                               | C                 | C                    | **A** | R               | C     |                                   |
| Manual k8s-deployments change out of hours (`emergency/`) | R                 | **A**                | I     | I               | I     | I                                 |
| Felet Lambda changes (Cloud Team account)                 | R                 | **A**                | I     | I               | I     | C                                 |
| Communication                                             | R                 | I                    | **A** | R               | I     |                                   |
| Postmortem                                                | R                 | **A**                | C     | C               | C     |                                   |

**Release captain and developer on duty:** who fills these roles is not decided yet. Proposal: one developer per sprint, in a fixed rotation, is both the release captain and the developer on duty. The schedule is kept on [Release to PRD](../Runbooks/Release-to-PRD.md). To decide in the alignment meeting.

**Gates:** PR pipeline green, one reviewer who is not the author, ACC with a smoke test by QA (never skipped), PRD approval.

**After the hotfix:** tag `release/<date>.1`, merge back to `main` the same day (H2), postmortem within 5 working days for Sev 1 and 2, a regression test, and a work item for the root cause.

**Metrics:** hotfixes per month, time from detection to PRD, failed hotfixes.

## Example: a bug in the BFF breaks claim processing

1. 10:05 Splunk alert: 5xx on the BFF above 2% for 10 minutes. Release `release/2026.10.20` went out at 09:30.
2. 10:10 Developer on duty and The Engineering Team declare Sev 1. The last release is less than 48 hours old, so they **roll back first**.
3. 10:25 Service restored on the previous release candidate.
4. 11:30 Fix merged to `main` (H1: `main` only contains the faulty release and the fix), new release candidate, ACC smoke test, PRD at 12:15, tagged `release/2026.10.20.1`.
5. Postmortem planned, regression test added.

## Acceptance criteria coverage

| Acceptance criterion                  | Where                                                                                                                                                                                                 |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Investigation document                | This page                                                                                                                                                                                             |
| Definition of a hotfix emergency      | Severity table                                                                                                                                                                                        |
| Workflow options and comparison       | Options, [ADR006](../Decisions/ADR006-Handle-PRD-incidents-with-a-hotfix-process.md)                                                                                                                  |
| Approval and escalation path          | Roles                                                                                                                                                                                                 |
| Validation gates                      | Gates                                                                                                                                                                                                 |
| Integration with the release strategy | Hotfixes produce release candidates and a `.1` tag ([ADR003](../Decisions/ADR003-Promote-release-candidates-to-PRD.md), [ADR004](../Decisions/ADR004-Tag-PRD-releases-and-generate-the-changelog.md)) |
| Integration with rollback             | Decision flow, [ADR007](../Decisions/ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md)                                                                                               |
| Communication plan                    | [Hotfix runbook](../Runbooks/Hotfix-runbook.md)                                                                                                                                                       |
| Post-hotfix process                   | After the hotfix, [Postmortem template](../Templates/Postmortem-template.md)                                                                                                                          |
| NX dependency analysis                | Findings (coupled pairs)                                                                                                                                                                              |
| On-call and availability              | Open: management decision                                                                                                                                                                             |
| Example scenario                      | Example                                                                                                                                                                                               |
| Recommendation                        | ADR006                                                                                                                                                                                                |
| Metrics                               | Metrics                                                                                                                                                                                               |
| Team alignment meeting                | Planned                                                                                                                                                                                               |

## Open questions

- Support outside office hours: none, best effort or on call? (management decision)
- Who is the release captain and who is the developer on duty, and how does the rotation work? Proposal: one developer per sprint does both. (to decide in the alignment meeting)
- Does a hotfix in PRD need a ServiceNow change?
