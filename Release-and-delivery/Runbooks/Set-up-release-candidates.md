# Set up release candidates

> **Draft implementation guide.** Depends on [ADR003](../Decisions/ADR003-Promote-release-candidates-to-PRD.md) and [ADR004](../Decisions/ADR004-Tag-PRD-releases-and-generate-the-changelog.md), both proposed. Use it to plan the stories under US 391474. The concept is explained in [Release candidates explained](../How-releases-work/Release-candidates-explained.md).

## What we build

| Where                                                                                                   | Change                                                                                                                                |
| ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| CI pipeline (`pipelines/ci-pipeline.yml`, definition 3415)                                              | Base = previous release candidate; publish a release candidate manifest; synthesize all CDK stacks and prepare all WSO2 configuration |
| CD pipeline (`pipelines/cd-pipeline.yml`, definition 3417, with `pipelines/templates/deploy-steps.yml`) | Guard against older release candidates; deploy only apps that differ; changelog before approval; completeness check; tag after PRD    |
| PR pipeline (definition 3418)                                                                           | Check the PR title format                                                                                                             |
| Azure DevOps settings                                                                                   | Branch control on ACC and PRD; "Create tag" permission for the build identity; run retention                                          |

## Before you start

- [ ] ADR003 and ADR004 are accepted.
- [ ] Quick wins merged: NX base commit fix (US 430950), CI never "partially succeeded" (US 441565), real commit in `UPDATE-K8S-DEPLOYMENTS.json` (`"sha": "$(Build.SourceVersion)"`).
- [x] The build identity has **Create tag** on the `nn-si-bic-claim-assistant` repository (not Contribute, no policy bypass). Checked 20 October 2026: allowed at project level and inherited. Keep Force push unset, so release tags cannot be moved or deleted by the pipeline.
- [ ] Image lifecycle rules in Artifactory and ECR keep at least 5 PRD versions.
- [ ] WSO2: republishing unchanged configuration is harmless (checked in TST).
- [ ] CDK: `cdk deploy --all` duration measured in TST, including the cross-account Felet Lambda.

## Step 1: the manifest format

Store the manifest as pipeline artifact `ReleaseManifest` (file `release-candidate.json`):

```json
{
  "rc": 559905,
  "commit": "4c86cff8",
  "createdAt": "2026-10-08T09:30:00Z",
  "apps": {
    "bic-claim-assistant-web": { "image": "559905", "builtFrom": "4c86cff8" },
    "bic-claim-assistant-api": { "image": "559905", "builtFrom": "4c86cff8" },
    "bic-document-retrieval-api": { "image": "549747", "builtFrom": "384f0878" },
    "bic-event-ingestion-api": { "image": "549747", "builtFrom": "384f0878" },
    "mockflow-api": { "image": "549747", "builtFrom": "384f0878", "prd": false }
  },
  "infra": {
    "event-ingestion-cdk": { "artifactsFromRun": 559905 },
    "felet-lambda-cdk": { "artifactsFromRun": 559905 }
  },
  "wso2": { "artifactsFromRun": 559905 }
}
```

## Step 2: CI on main and hotfix branches

1. **Find the previous release candidate:** download `ReleaseManifest` from the newest successful CI run on the same branch that published one. Fallback: the newest `release/*` tag, then `HEAD~1`.
2. **Set the base:** `BASE_SHA = previous.commit`. A failed run publishes no release candidate, so its changes are included in the next one automatically.
3. **Build:** images for `nx show projects --affected --base=$BASE_SHA --withTarget ci-docker-build-and-push`, or the explicit `deployProjects` list.
4. **Write the manifest:** take the app list from NX (`nx show projects --withTarget ci-docker-build-and-push`), so new apps are added and removed apps are dropped. Rebuilt apps get the new build id; the other apps keep their entry from the previous manifest. Set `commit = $(Build.SourceVersion)`.
5. **Infra and WSO2:** synthesize **all** CDK stacks and prepare **all** WSO2 configuration, so every release candidate can be deployed on its own.
6. **Publish:** `ReleaseManifest`, `CDKArtifact`, `WSO2Artifact`.

## Step 3: CD, one run per release candidate

For each stage (TST, ACC, PRD):

1. **Guard:** read the commit the environment runs (from k8s-deployments `values.json`, after quick win N3). If `git merge-base --is-ancestor <rc commit> <environment commit>` shows the environment is newer, stop with "Environment already runs a newer release candidate". Skip the check when the run parameter `ROLLBACK=true`.
2. **Apps:** compare each app in the manifest with `apps/nn-si-bic-claim-assistant/<app>/<env>/values.json`. Call the `update-deployment` pipeline (definition 1401) only for apps whose image tag differs, with `HEAD_SHA = builtFrom`. Skip `mockflow-api` in PRD.
3. **CDK:** `cdk deploy --all` with the `CDKArtifact` of the release candidate. CloudFormation skips unchanged stacks.
4. **WSO2:** publish all configuration from `WSO2Artifact`.
5. **Completeness check:** every app in the environment now equals the manifest. Show "N merges on `main` not in this release candidate" in the summary.

At the TST stage, add the **changelog** as a run summary tab:
`nx release changelog <version> --from <commit PRD runs> --to <rc commit> --dry-run`, uploaded with `##vso[task.uploadsummary]`. Read the PRD commit from `apps/nn-si-bic-claim-assistant/<app>/prd/values.json` (after quick win N3), not from the newest tag: after a rollback they differ. Add the app version differences per environment.

After a successful **ACC** stage, add a retention lease of 30 days to the run, so a release candidate that reached ACC but not PRD can still be released later (for example after a rollback).

After a successful **PRD** stage:

1. Create the annotated tag `release/YYYY.MM.DD` (add `.N` if the tag exists) on the release candidate commit, with the changelog as message.
2. Add a retention lease to the run, so artifacts stay for rollback.
3. Post a Teams message with the tag and the changed apps.

## Step 4: Azure DevOps settings

- ACC and PRD environments: **branch control** allows only `main` and `hotfix/*`.
- CD starts automatically only for `main` and `hotfix/*`.
- PR pipeline: fail when the PR title does not match `type(#ticket): description`.

## Step 5: bootstrap

1. Create the first manifest by hand from today's k8s-deployments PRD state (image tags per app) and publish it as `ReleaseManifest` in a one-off pipeline run.
2. Tag `release/2026.09.30` on `384f0878`: what PRD really runs today.
3. From the next merge on, CI builds release candidates automatically.

## Step 6: verify in TST and ACC

| Scenario                                                                                         | Expected                                         |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------ |
| Merge PRs that change the BFF, the Portal, then the BFF; promote only the last release candidate | All three changes reach ACC                      |
| Start the CD run of an older release candidate                                                   | The guard stops it                               |
| Same older release candidate with `ROLLBACK=true`                                                | Deploys; only the changed apps restart           |
| A CI run fails, the next one succeeds                                                            | The next release candidate contains both changes |
| A documentation only PR                                                                          | New release candidate, no app redeployed         |
| Successful PRD stage (in ACC: dry run of the tag step)                                           | Correct tag name and changelog                   |

## Audit: where to find answers

| Question                             | Source                                                                                           |
| ------------------------------------ | ------------------------------------------------------------------------------------------------ |
| Which version runs in environment X? | k8s-deployments `apps/nn-si-bic-claim-assistant/<app>/<env>/values.json` (`image.tag`, `commit`) |
| Which release is in PRD?             | The newest `release/*` tag and its release candidate                                             |
| What changed between two releases?   | `git log release/A..release/B`, changelog in the tag                                             |
| Who approved, and when?              | Approvals of the CD run (kept with a retention lease)                                            |
| Which stories were delivered?        | Work items linked to the PRs in the range, release notes page                                    |
| Which infrastructure version?        | Release candidate, CI run, `cdk.out`, CloudFormation events                                      |

## Later

- A release pipeline with a release candidate picker and a dashboard with the versions in TST, ACC and PRD.
- DORA metrics: deployment frequency, lead time, change failure rate, time to restore.
