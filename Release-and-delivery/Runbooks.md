# Runbooks

Step by step instructions for the moment you need them.

> **Drafts.** Each runbook depends on an ADR that is still proposed. It becomes final when that ADR is accepted.

| Runbook                                                                  | Use it when                                        | Depends on     |
| ------------------------------------------------------------------------ | -------------------------------------------------- | -------------- |
| [Release to PRD](./Runbooks/Release-to-PRD.md)                           | You are the release captain for a PRD release      | ADR003, ADR004 |
| [Hotfix runbook](./Runbooks/Hotfix-runbook.md)                           | PRD has a serious problem that cannot wait         | ADR006         |
| [Rollback runbook](./Runbooks/Rollback-runbook.md)                       | The last release broke PRD and you need to go back | ADR007         |
| [Set up release candidates](./Runbooks/Set-up-release-candidates.md)     | You build the release candidate pipeline           | ADR003, ADR004 |
| [Use the preview environment](./Runbooks/Use-the-preview-environment.md) | You want to show or test a feature branch          | ADR005         |
