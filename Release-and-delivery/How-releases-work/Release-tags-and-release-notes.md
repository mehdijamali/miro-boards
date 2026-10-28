# Release tags and release notes

> **Status: proposed** in [ADR004](../Decisions/ADR004-Tag-PRD-releases-and-generate-the-changelog.md).

Every PRD release gets three things: a **release tag** that marks it, a **changelog** that lists the technical changes, and **release notes** that explain it to people.

## Release tag

- Format: `release/YYYY.MM.DD`, for example `release/2026.10.20`.
- A second release on the same day, or a hotfix, adds a number: `release/2026.10.20.1`.
- The pipeline places the tag on the commit of the release candidate, right after a successful PRD deployment. It is an annotated tag that also holds the changelog.
- Images keep their own build id tag, for example `559905`. The release candidate links the release tag to the image of every app.

What you can do with tags:

| Question                           | How                                                                                  |
| ---------------------------------- | ------------------------------------------------------------------------------------ |
| What is in PRD?                    | The commit in k8s-deployments for PRD, and its release tag (normally the newest one) |
| What changed between two releases? | `git log release/2026.10.06..release/2026.10.20`                                     |
| Where does a hotfix branch start?  | From the release tag of what PRD runs                                                |

The first tag will be `release/2026.09.30` on commit `384f0878`: what PRD really runs today. That gives the first generated changelog a correct starting point.

Tags are never moved or deleted. After a rollback, the newest tag is no longer what PRD runs, until the next release gets its own tag. That is why the changelog starts from the commit PRD runs, not from the newest tag.

## Changelog

The changelog is generated, not written. It comes from the **PR titles**, because every PR is squash merged and its title becomes the commit message on `main`.

**PR title format** (Conventional Commits, the NonLife standard in ITSD EDR 00003):

`type(#ticket): what changed`

| Type                                               | Use for                | Changelog section |
| -------------------------------------------------- | ---------------------- | ----------------- |
| `feat`                                             | New functionality      | Features          |
| `fix`                                              | A bug fix              | Fixes             |
| `perf`                                             | Faster, same behaviour | Performance       |
| `refactor`, `chore`, `build`, `ci`, `docs`, `test` | Everything else        | Other             |

Examples:

| ✅ Good                                     | ❌ Missing from the changelog                 |
| ------------------------------------------- | --------------------------------------------- |
| `feat(#418043): show timeline annotations`  | `Feat/#418043 implement timeline annotations` |
| `fix(#395859): send no-store cache headers` | `Feature: Implement Automatic Summary 2.0`    |

Since April, 77% of our PR titles already follow the format. The PR pipeline will check the title, so the rest follows.

**Where you see it:** in the CD pipeline run, as a summary tab, **before** you approve ACC. It lists every PR between the commit PRD runs and the release candidate (Compass and Handler Portal do the same with `nx release changelog --dry-run`). After the PRD release it is also stored in the release tag.

## Release notes

Release notes are short and written for people: claim handlers, the PO and stakeholders.

- **Owner:** the PO, helped by the release captain.
- **Where:** one wiki page per PRD release, from the [Release notes template](../Templates/Release-notes-template.md).
- **Content:** what users notice, in two to five bullets; known issues; links to the changelog and work items.
- **When:** published on the day of the PRD release.

Linked work items can pre-fill the draft: every PR is linked to a user story or bug, so the list of stories in a release can be generated from the PRs since the last tag.

## Related pages

- [Release candidates explained](./Release-candidates-explained.md)
- [Release to PRD](../Runbooks/Release-to-PRD.md)
- [ADR004 Tag PRD releases and generate the changelog](../Decisions/ADR004-Tag-PRD-releases-and-generate-the-changelog.md)
