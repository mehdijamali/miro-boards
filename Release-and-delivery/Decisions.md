# Decisions

Architecture decision records (ADRs) for how we release. Each page describes one decision, the options we considered, their pros and cons, and the proposal.

## How we decide

We follow the team level process of our [ADR guide](<https://dev.azure.com/NonLife/Zakelijk%20Schade/_git/nn-si-bic-claim-assistant?path=/docs/adr/Architecture%20decision%20record%20(ADR).md>):

1. The owner shares the ADR with the team (status **proposed**).
2. Team members add comments. The owner turns comments into action points and resolves them.
3. Each decider votes **accepted** or **rejected**.
4. The owner sets the status and the date. An accepted ADR does not change anymore; a new ADR supersedes it.

All tools in these ADRs are already used in NN, so the decisions are team scope.

## The decisions

| ADR                                                                                     | Decision                                                | Status   |
| --------------------------------------------------------------------------------------- | ------------------------------------------------------- | -------- |
| [ADR003](./Decisions/ADR003-Promote-release-candidates-to-PRD.md)                       | Promote release candidates to PRD                       | proposed |
| [ADR004](./Decisions/ADR004-Tag-PRD-releases-and-generate-the-changelog.md)             | Tag PRD releases and generate the changelog             | proposed |
| [ADR005](./Decisions/ADR005-Test-feature-branches-in-a-preview-environment.md)          | Test feature branches in a preview environment          | proposed |
| [ADR006](./Decisions/ADR006-Handle-PRD-incidents-with-a-hotfix-process.md)              | Handle PRD incidents with a hotfix process              | proposed |
| [ADR007](./Decisions/ADR007-Roll-back-by-redeploying-the-previous-release-candidate.md) | Roll back by redeploying the previous release candidate | proposed |

ADR001 (secure Portal to BFF authentication) and ADR002 (pnpm catalog, PR 100291) live in the repository under `docs/adr`. Numbering continues from there.
