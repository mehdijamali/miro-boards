# Use the preview environment

> **Draft.** Depends on [ADR005](../Decisions/ADR005-Test-feature-branches-in-a-preview-environment.md) (proposed). The preview environment does not exist yet; the setup tasks are at the end of this page.

## What it is

An extra AI Portal and BFF in our TST namespace that run **one feature branch** at a time:

- Portal: `bic-claim-assistant-web-preview.nonlife-si-t.nonlife-si-t-k8s.nn-cloud.net`
- BFF: `bic-claim-assistant-api-preview.nonlife-si-t.nonlife-si-t-k8s.nn-cloud.net`

TST itself always runs `main`. The preview uses the same TST backends: AI Brain TST, FELET TST, the shared CDK stacks.

## When to use it

- UX/UI or the PO wants to review a screen before the PR is merged.
- You want to test a Portal or BFF change against TST data without disturbing TST.

Do **not** use it for CDK, Lambda or WSO2 changes: those are not part of the preview and stay shared.

## How to use it

1. **Book it** in the team channel: "Preview: `feature/<branch>` until [time], [your name]". One branch at a time.
2. **Deploy:** in Azure DevOps run the pipeline **bic-claim-assistant-preview** and select your branch. It builds the Portal and BFF images and updates the `-preview` apps in TST.
3. **Share the link** of the preview Portal with UX or the PO.
4. **Release it** in the channel when you are done.

## Rules

- Feature branches never go to TST, ACC or PRD; the CD pipeline only starts automatically for `main` and `hotfix/*`.
- The preview runs TST data. Do not use it for load tests.
- If your branch changes the shared contract, deploy the Portal and the BFF together (the preview pipeline always builds both).

## Setup tasks (for the team that builds it)

- [ ] Pipeline `pipelines/preview-pipeline.yaml`, based on the BIC Portal pipeline (`nn-si-bic/pipelines/preview-pipeline.yaml`): manual trigger, builds web and api from the chosen branch, calls `update-deployment` for the `-preview` apps in `tst` only.
- [ ] Two apps in k8s-deployments: `apps/nn-si-bic-claim-assistant/bic-claim-assistant-web-preview/tst` and `.../bic-claim-assistant-api-preview/tst`. Changes in `tst/` need no Cloud Platform Team review; the Helm chart creates the addresses.
- [ ] Azure AD: add the redirect URI of the preview Portal to our app registration.
- [ ] Preview Portal: `API_BASE_URL` points to the preview BFF; `NEXTAUTH_URL` to the preview Portal.
- [ ] Preview BFF: `ALLOWED_ORIGINS` includes the preview Portal address.
- [ ] Secrets for both apps in AWS Secrets Manager (`nn-si-bic-claim-assistant/tst/...-preview`).
- [ ] Branch control on ACC and PRD, and CD auto start only for `main` and `hotfix/*`.

## Related pages

- [ADR005 Test feature branches in a preview environment](../Decisions/ADR005-Test-feature-branches-in-a-preview-environment.md)
- [Release candidates explained](../How-releases-work/Release-candidates-explained.md)
