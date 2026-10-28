# US 391474 PRD release orchestration

|           |                                                                                                                                                                                                                                   |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Spike     | [US 391474: 1. Spike - PRD release orchestration](https://dev.azure.com/NonLife/Synera%202.0/_workitems/edit/391474)                                                                                                              |
| Epic      | [439240 Techdebt 2026](https://dev.azure.com/NonLife/Synera%202.0/_workitems/edit/439240)                                                                                                                                         |
| Status    | Draft for the alignment meeting                                                                                                                                                                                                   |
| Data      | Collected from Azure DevOps on 15 October 2026                                                                                                                                                                                    |
| Decisions | [ADR003](../Decisions/ADR003-Promote-release-candidates-to-PRD.md), [ADR004](../Decisions/ADR004-Tag-PRD-releases-and-generate-the-changelog.md), [ADR005](../Decisions/ADR005-Test-feature-branches-in-a-preview-environment.md) |
| Related   | US 430950 (NX base commit), US 441565 (Sonar quality gate), Task 383452 (PRD deploy strategy), US 390559 (release process), US 443029 and ADR002 (pnpm catalog)                                                                   |

## The question

How do we make sure a PRD release contains **everything** merged to `main`, never goes back by accident, and can be explained afterwards?

## Summary

Today a PRD release is whatever one pipeline run built: the apps NX marked as affected in that run. Changes are left behind when approvals time out, approving an older run can silently roll PRD back, and nobody can say what runs in PRD. We propose **release candidates** ([ADR003](../Decisions/ADR003-Promote-release-candidates-to-PRD.md)): every merge produces a manifest with the version of every app, and we promote that manifest. Plus release tags and a generated changelog ([ADR004](../Decisions/ADR004-Tag-PRD-releases-and-generate-the-changelog.md)), a preview environment so TST stays `main` ([ADR005](../Decisions/ADR005-Test-feature-branches-in-a-preview-environment.md)), and eight quick wins we can do now.

## How it works today

What we deploy:

| Deployable                   | What it is                             | How it is deployed                 | Environments  |
| ---------------------------- | -------------------------------------- | ---------------------------------- | ------------- |
| `bic-claim-assistant-web`    | AI Portal (Next.js)                    | Image, k8s-deployments, ArgoCD     | TST, ACC, PRD |
| `bic-claim-assistant-api`    | BFF (NestJS)                           | same                               | TST, ACC, PRD |
| `bic-document-retrieval-api` | Claim documents from FELET             | same                               | TST, ACC, PRD |
| `bic-event-ingestion-api`    | Receives FELET events                  | same                               | TST, ACC, PRD |
| `mockflow-api`               | Test double                            | same                               | TST, ACC      |
| `event-ingestion-cdk`        | AWS resources and Lambda for ingestion | `cdk deploy` from the CD pipeline  | TST, ACC, PRD |
| `felet-lambda-cdk`           | FELET Lambda in the Cloud Team account | `cdk deploy`, cross-account        | TST, ACC, PRD |
| WSO2 configuration           | API gateway definitions                | `wso2wrapper` from the CD pipeline | TST, ACC, PRD |

Two pairs share an API contract and must move together when it changes: the Portal with the BFF, and the event ingestion API with its CDK stack.

::: mermaid
graph LR
PR["PR pipeline"] -->|"squash merge"| MAIN["main"]
MAIN --> CI["CI: build affected apps"]
CI --> TST["TST, automatic"]
TST --> ACC["ACC, approval, times out after 24 h"]
ACC --> PRD["PRD, approval, times out after 24 h"]
PRD --> GITOPS["k8s-deployments, ArgoCD"]
:::

- One CD run per CI run, carrying only that run's affected apps.
- The NX base commit is the last CI run whose result is exactly `succeeded`.
- When the ACC approval times out, ACC and PRD are skipped for that run for good.
- The PRD approval also times out after 24 hours since 21 October 2026. Before, it waited up to 30 days, so an old run could still be approved at PRD after a newer one and roll PRD back.
- Any branch can trigger CD into TST; ACC and PRD have no branch control.
- k8s-deployments PRs created by the pipeline complete in under a minute; manual PRs for ACC and PRD need Cloud Platform Team review unless the branch starts with `emergency/`.

## Findings

| #   | Finding                                                                                                                         | Evidence                                                                                                                                                                                                                                                                                              |
| --- | ------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| F1  | **Changes are left behind.** A release is one run's affected set.                                                               | Only 10 of 39 runs from `main` reached PRD; 23 timed out at ACC. Automatic Summary 2.0 (29 September) reached PRD only because the next approved run rebuilt every app. On 29 September two runs waited at ACC at the same time; approving them in the wrong order would have removed a fix from PRD. |
| F2  | **The NX base gets stuck.** Sonar may fail without failing the run, so runs end "partially succeeded" and do not count as base. | 21 of 42 CI runs on `main` ended partially succeeded. The base stayed on one commit for 6 merges (25 August to 7 September) and for 5 merges (9 to 29 September). A documentation only PR still redeployed the event ingestion API.                                                                   |
| F3  | **Dependency updates redeploy everything.** A lock file change marks all projects as affected (Nx default).                     | PRs 93574, 99062 and 99088 rebuilt all 5 apps and both CDK stacks. The NX setting `"auto"` is not safe on Nx 22.7 (it would miss lock file only updates); ADR002 keeps `"all"` until the Nx 23 upgrade.                                                                                               |
| F4  | **Workarounds are needed to deploy.**                                                                                           | PR 93657 added comments only to force a redeploy. On 13 July branch `deploy/414448-bic-document-retrieval-api` went to ACC and PRD; its commit never reached `main`.                                                                                                                                  |
| F5  | **What runs in PRD is unclear.** k8s-deployments records the NX base commit, not the built commit; there are no release tags.   | PRD says `af154d0`; it runs an image built from `384f0878`.                                                                                                                                                                                                                                           |
| F6  | **No changelog and no release notes.**                                                                                          | 78 merges since 15 April: 77% of PR titles follow Conventional Commits, but the biggest features do not. The PR title is not checked.                                                                                                                                                                 |
| F7  | **TST is also the playground.**                                                                                                 | 31 of 72 CD runs deployed a feature branch to TST. No branch control on ACC and PRD.                                                                                                                                                                                                                  |

## Options and proposal

| Topic                        | Options                                                                                                                                   | Proposal                                                   | Decision page                                                                   |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Release model                | A keep as is · B since last release tag · C release candidates · D release branches · E deploy everything · F pipeline per app            | **C** (B as interim)                                       | [ADR003](../Decisions/ADR003-Promote-release-candidates-to-PRD.md)              |
| Release marker and changelog | Date tag · semantic version · build id · none; changelog file · changelog per stage · wiki release notes · work item draft · change files | **Date tag, changelog per stage, release notes by the PO** | [ADR004](../Decisions/ADR004-Tag-PRD-releases-and-generate-the-changelog.md)    |
| Feature branches             | TST etiquette · preview environment · DEV · per PR                                                                                        | **Preview environment in TST**                             | [ADR005](../Decisions/ADR005-Test-feature-branches-in-a-preview-environment.md) |
| Release rhythm               | When valuable, at least every sprint · weekly · sprint end                                                                                | **When valuable, at least every sprint**                   | Team agreement in [Release to PRD](../Runbooks/Release-to-PRD.md)               |

Explanation for the whole team: [Release candidates explained](../How-releases-work/Release-candidates-explained.md).

## Quick wins (useful whatever we decide)

| #   | Change                                                                                     | Fixes | Size | Work item             |
| --- | ------------------------------------------------------------------------------------------ | ----- | ---- | --------------------- |
| N1  | NX base = the last CI run that published its artifacts (later: the previous RC)            | F2    | S    | US 430950             |
| N2  | Sonar must not turn a CI run "partially succeeded"                                         | F2    | S    | US 441565             |
| N3  | Record the real commit: `"sha": "$(Build.SourceVersion)"` in `UPDATE-K8S-DEPLOYMENTS.json` | F5    | XS   | new                   |
| N4  | Merge the pnpm catalog (ADR002); keep `"all"` until the Nx 23 upgrade gate                 | F3    | S    | US 443029             |
| N5  | A `deployProjects` CI parameter instead of dummy commits                                   | F4    | S    | new                   |
| N6  | Branch control on ACC and PRD (`main`, `hotfix/*`)                                         | F7    | XS   | new                   |
| N7  | Check PR titles in the PR pipeline                                                         | F6    | S    | new                   |
| N8  | CD starts automatically only for `main` and `hotfix/*`                                     | F7    | XS   | new                   |
| N9  | PRD approval timeout from 30 days to 24 hours, like ACC                                    | F1    | XS   | done, 21 October 2026 |

## How other SI teams do it

There is no mandatory SI release process. The common building blocks:

| Team                                        | What we can reuse                                                                                                     |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Score                                       | Last release tag as NX base (same problem as F1), semantic version tags, hotfix from a tag, release notes on the wiki |
| Compass and Handler Portal                  | Changelog per stage in the pipeline summary; Compass tags `release/YYYYMMDD.N.0`                                      |
| Claims Portal and Claim Registration Portal | A wiki page per release with a ServiceNow change; only the stable branch reaches ACC and PRD                          |
| BIC Portal                                  | Preview environment `bic-portal-preview` in TST                                                                       |
| Inkomen platform                            | Only deploys a version that already runs in the previous environment                                                  |
| ITSD EDR 00003                              | Conventional Commits is the NonLife standard                                                                          |

## Rollout plan

| Phase | Content                                                                            | Work items                        |
| ----- | ---------------------------------------------------------------------------------- | --------------------------------- |
| 0     | Quick wins N1 to N8                                                                | US 430950, US 441565, new stories |
| 1     | Release candidates: manifest, guard, comparison, changelog, release tag, retention | New stories under US 391474       |
| 2     | Preview environment                                                                | New story                         |
| 3     | Hotfix and rollback runbooks final, first rollback drill                           | US 441563, US 441564              |
| 4     | Optional: release dashboard, DORA metrics                                          | none yet                          |

Migration: create the first release candidate from today's k8s-deployments state and tag `release/2026.09.30` on `384f0878`. Implementation guide: [Set up release candidates](../Runbooks/Set-up-release-candidates.md).

## Acceptance criteria coverage

| Acceptance criterion                                         | Where                                                                                                                                               |
| ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Investigation document covering all decision points          | This page, ADR003 to ADR005                                                                                                                         |
| Current release process mapped                               | How it works today, F1, F7                                                                                                                          |
| Architecture diagram per proposed strategy                   | This page, [Release candidates explained](../How-releases-work/Release-candidates-explained.md)                                                     |
| NX affected logic analysed                                   | F2, F3, quick wins                                                                                                                                  |
| Git and Azure DevOps capabilities (tags, ranges, automation) | [ADR004](../Decisions/ADR004-Tag-PRD-releases-and-generate-the-changelog.md), [Set up release candidates](../Runbooks/Set-up-release-candidates.md) |
| Risk assessment per approach                                 | [ADR003](../Decisions/ADR003-Promote-release-candidates-to-PRD.md), open questions                                                                  |
| Release completeness validation                              | Guard and completeness check in [Set up release candidates](../Runbooks/Set-up-release-candidates.md)                                               |
| Audit and traceability                                       | Audit table in [Set up release candidates](../Runbooks/Set-up-release-candidates.md)                                                                |
| Recommendation with pros and cons                            | ADR003                                                                                                                                              |
| Rollout and migration plan                                   | Rollout plan                                                                                                                                        |
| Related work items                                           | Header, quick wins                                                                                                                                  |
| Team alignment meeting                                       | Planned, with the presentation                                                                                                                      |

## Open questions

- ~~Permission: the build identity needs "Create tag" on our repository.~~ **Answered (checked 20 October 2026):** "Zakelijk Schade Build Service" already has Create tag, set at project level and inherited by our repository. Force push is deliberately not set, so the pipeline cannot move or delete release tags.
- WSO2: is republishing unchanged configuration harmless (API revisions)?
- CDK: how long does `cdk deploy --all` take per release, including the cross-account Felet Lambda?
- Image retention in Artifactory and ECR: do lifecycle rules keep at least 5 PRD versions?
- ServiceNow: does a PRD release need a change record? (PO to confirm)
- ACC approval: today the person who started the run may approve ACC (PRD needs someone else). Do we want a second person for ACC too?
- Work that depends on AI Brain must not be merged before both sides are ready.
