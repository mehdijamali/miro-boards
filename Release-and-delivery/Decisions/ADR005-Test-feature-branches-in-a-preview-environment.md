# ADR005 Test feature branches in a preview environment

- Status: proposed
- Deciders: BIC team (Engineers, PO)
- Date: 2026-10-19
- Scope: team
- PR: none yet; reviewed on the wiki and in the alignment meeting

## Context and Problem Statement

This is a team level decision about where we test and show work that is not merged yet.

Between mid-June and 15 October 2026, 31 of 72 CD runs deployed a **feature branch** into the shared TST environment. So TST often runs code that is not in `main`, and a PO or tester may accept a different build than the one we release. There is also no branch control on ACC and PRD: a `fix/` branch reached ACC and a `deploy/` branch reached PRD; its commit never reached `main`.

**Question:** where do developers, UX and the PO try unfinished work, while TST, ACC and PRD only get `main`?

## Decision Drivers

- TST always runs `main`, so acceptance in TST means something.
- UX/UI, front-end and the PO can review a branch before it is merged.
- Only `main` and `hotfix/*` can reach ACC and PRD.
- Small effort, and as little Cloud Platform Team work as possible.

## Considered Options

- E1. TST etiquette: book TST in the channel and deploy `main` back afterwards
- E2. A preview environment in TST: extra Portal and BFF apps for one branch
- E3. A new DEV environment
- E4. A temporary environment per PR

## Pros and Cons of the Options

### E1. TST etiquette

- Good, because it is free.
- Bad, because people forget, and TST still often differs from `main`.

### E2. Preview environment in TST

Two extra apps in our TST namespace, `bic-claim-assistant-web-preview` and `bic-claim-assistant-api-preview`, deployed from any branch by a manual preview pipeline. The BIC Portal team already does this with `bic-portal-preview`.

- Good, because TST stays equal to `main`.
- Good, because UX and the PO can review a branch on a stable address.
- Good, because changes in the `tst/` folder of k8s-deployments need no Cloud Platform Team review, and the Helm chart creates the address automatically.
- Bad, because it covers the Portal and the BFF only; CDK and Lambda changes still share TST.
- Bad, because the preview address needs its own Azure AD redirect URI and `API_BASE_URL`.
- Bad, because one preview environment serves one branch at a time.

### E3. New DEV environment

- Good, because it isolates completely.
- Bad, because it needs the Cloud Platform Team (folders, ArgoCD, cluster mapping), AWS resources, secrets and WSO2 configuration.
- Bad, because upstream systems (AI Brain, FELET) may have no DEV environment.

### E4. Environment per PR

- Good, because every PR gets its own environment.
- Bad, because it needs a lot of Cloud Platform Team work and wildcard DNS and TLS.
- Bad, because Azure AD strongly discourages wildcard redirect URIs, so every PR address needs its own redirect URI.

## Decision Outcome

Proposed option: **"E2. Preview environment in TST"**, because it keeps TST equal to `main` at a small cost, using a pattern that already works in our domain. Together with it:

- Branch control on the ACC and PRD environments: only `main` and `hotfix/*`.
- The CD pipeline only starts automatically for `main` (and `hotfix/*`).

We revisit a DEV or per PR environment later if one preview environment is not enough. How to use it: [Use the preview environment](../Runbooks/Use-the-preview-environment.md).

## Positive Consequences

- TST is always `main`, so acceptance in TST is meaningful.
- No more feature branches in ACC or PRD.
- UX and the PO get a place to review work in progress.

## Negative Consequences

- One branch at a time: the team agrees who uses it.
- Setup work: preview pipeline, two apps in k8s-deployments, Azure AD redirect URI.
- Changes to CDK stacks or Lambdas still need care in the shared TST.

## Prepared by

- BIC Claim Assistant team, from spike US 391474

## Related ADR

- [ADR003 Promote release candidates to PRD](./ADR003-Promote-release-candidates-to-PRD.md)
