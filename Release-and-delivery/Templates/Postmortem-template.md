# Postmortem template

> Copy this page to a new page named **Postmortem YYYY.MM.DD short title** and fill it in within 5 working days after a Sev 1 or Sev 2 incident. Owner: the developer on duty, reviewed by The Engineering Team. Blameless: we look for causes in the process and the system, not in people.

---

# Postmortem YYYY.MM.DD short title

|                    |                                          |
| ------------------ | ---------------------------------------- |
| Severity           | Sev 1 or Sev 2                           |
| Detected           | date and time, by whom or by which alert |
| Resolved           | date and time                            |
| Time to restore    | minutes                                  |
| Users affected     | who and how many, what they saw          |
| Related release    | `release/YYYY.MM.DD`                     |
| Resolution         | rollback, hotfix (H1, H2 or H4) or other |
| Incident work item | link                                     |

## Summary

Three to five sentences: what happened, the impact, and how we fixed it.

## Timeline

| Time  | What happened              |
| ----- | -------------------------- |
| HH:MM | Alert or first report      |
| HH:MM | Severity declared          |
| HH:MM | Rollback or hotfix started |
| HH:MM | Service restored           |
| HH:MM | Final fix in PRD           |

## Root cause

What caused it, and why our checks (tests, review, ACC) did not catch it.

## What went well

- ...

## What can be better

- ...

## Actions

| Action          | Owner | Work item | Due |
| --------------- | ----- | --------- | --- |
| Regression test |       |           |     |
|                 |       |           |     |

## Runbook updates

What we changed in the [Hotfix runbook](../Runbooks/Hotfix-runbook.md) or the [Rollback runbook](../Runbooks/Rollback-runbook.md).
