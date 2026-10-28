# ADR004 Tag PRD releases and generate the changelog

- Status: proposed
- Deciders: BIC team (Engineers, PO)
- Date: 2026-10-19
- Scope: team
- PR: none yet; reviewed on the wiki and in the alignment meeting

## Context and Problem Statement

This is a team level decision about how we mark PRD releases and how we explain what changed.

Today there are no release tags (the only tag in the repository is `pentest-PTREQ0012033`), no changelog and no release notes. Since 15 April 2026, 78 PRs were merged; 60 (77%) of their titles follow Conventional Commits exactly, but the biggest features do not, for example "Feature: Implement Automatic Summary 2.0". The PR title is what lands on `main`, and it is not checked.

**Question:** how do we mark a PRD release, and how do approvers and users learn what is in it?

## Decision Drivers

- Everyone, also the PO, can read which release runs in PRD.
- Approvers see what they approve **before** they approve.
- Little manual work and no extra rights for the pipeline beyond creating a tag.
- Follows the NonLife standard for commit messages (Conventional Commits, ITSD EDR 00003).

## Considered Options

Release marker:

- M1. Date based tag (CalVer): `release/2026.10.20`, hotfix `release/2026.10.20.1`
- M2. Semantic version with `nx release`: `v1.4.0`
- M3. Build id only, as today: `549747`
- M4. No marker, use the pipeline run history

Changelog and release notes:

- C1. `CHANGELOG.md` committed to the repository by `nx release`
- C2. Changelog per stage in the pipeline run summary, generated before approval
- C3. Business release notes on the wiki, written by the PO
- C4. A draft generated from the work items linked to the PRs
- C5. A change file per PR (version plans, changesets)

## Pros and Cons of the Options

### M1. Date based tag

- Good, because everybody understands "the release of 20 October".
- Good, because there are no discussions about major or minor versions.
- Good, because it is easy to automate and sorts naturally.
- Bad, because it says nothing about breaking changes. We have no external consumers, and our API contracts are versioned separately.

### M2. Semantic version

- Good, because it signals breaking changes. The Score team uses it (`v2.0.0`).
- Bad, because it needs strict commit discipline and a release commit to bump `package.json`.
- Bad, because "major" is hard to define for a UI.

### M3. Build id only

- Good, because it exists.
- Bad, because it identifies one pipeline run, not a release.

### M4. Pipeline history

- Bad, because it is invisible in Git and disappears with run retention.

### C1. CHANGELOG.md in the repository

- Good, because the changelog lives with the code (Score, Compass, Retail Portal).
- Bad, because the pipeline must push to `main`, which needs a policy bypass, or someone must create a release PR.

### C2. Changelog per stage in the pipeline

- Good, because approvers see exactly what they approve (Compass, Handler Portal).
- Good, because the pipeline does not write to `main`.
- Bad, because it is not browsable in the repository; we store it in the annotated release tag.

### C3. Release notes on the wiki

- Good, because they are written for people and can link the ServiceNow change, tests and stories (Claims Portal, Claim Registration Portal, Score).
- Bad, because they are manual work and need an owner.

### C4. Draft from linked work items

- Good, because it pre-fills the release notes.
- Bad, because it is only as good as the linking of PRs to work items.

### C5. A change file per PR

- Bad, because it adds a file to every PR, which is friction for little gain at our size.

## Decision Outcome

Proposed: **M1 + C2 + C3**, with **C4** to pre-fill the release notes.

- Tag format `release/YYYY.MM.DD[.N]`, an annotated tag that holds the changelog, placed by the pipeline after a successful PRD deployment. The build identity only needs the "Create tag" permission, which it already has (checked 20 October 2026). Force push stays unset, so the pipeline can never move or delete a release tag.
- The changelog is generated with `nx release changelog --dry-run` and shown in the CD run before the ACC approval. It starts from the **commit PRD runs** (read from k8s-deployments, after quick win N3), not simply from the newest tag: after a rollback, the newest tag is no longer what PRD runs.
- The PO writes short release notes on the wiki for every PRD release, from the [Release notes template](../Templates/Release-notes-template.md).
- PR titles must follow `type(#ticket): description`; the PR pipeline checks it.
- The first tag is `release/2026.09.30` on `384f0878`, what PRD runs today.
- Tags are never moved or deleted. After a rollback, the tag of the release we rolled back from stays as history; a release candidate that reaches PRD later gets its own new tag at that moment.

Details: [Release tags and release notes](../How-releases-work/Release-tags-and-release-notes.md).

## Positive Consequences

- "What is in PRD and what changed?" is answered by the commit PRD runs, its release tag and the changelog.
- Approvers know what they approve.
- Hotfix branches have a clear starting point: the release tag of what PRD runs.

## Negative Consequences

- A PR title check adds a small rule for every PR.
- Release notes are a new task for the PO each release.
- C1 can still be added later if the team wants a changelog file in the repository.

## Prepared by

- BIC Claim Assistant team, from spike US 391474

## Related ADR

- [ADR003 Promote release candidates to PRD](./ADR003-Promote-release-candidates-to-PRD.md)
- [ADR006 Handle PRD incidents with a hotfix process](./ADR006-Handle-PRD-incidents-with-a-hotfix-process.md)
