# Release to PRD

> **Draft.** Depends on [ADR003](../Decisions/ADR003-Promote-release-candidates-to-PRD.md) and [ADR004](../Decisions/ADR004-Tag-PRD-releases-and-generate-the-changelog.md), both proposed. It describes releasing with release candidates. For today, see "Until release candidates exist" at the end.

## Who

| Role                                           | Does                                                                                                   |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| **Release captain** (a developer, in rotation) | Picks the release candidate, checks the changelog, runs the pipeline, approves ACC, posts the messages |
| **QA**                                         | Tests the release candidate in TST, runs the smoke test in ACC and PRD, reports the result             |
| **PO**                                         | Decides "go", approves PRD, writes the release notes                                                   |
| **The Engineering Team**                       | Approves PRD when the PO is not available; decides on rollback with the release captain                |

## Team agreements

| Topic                              | Proposal                                                                     | Status                             |
| ---------------------------------- | ---------------------------------------------------------------------------- | ---------------------------------- |
| How often we release to PRD        | When valuable, at least once per sprint                                      | to decide in the alignment meeting |
| Release window                     | Office hours, not on Friday afternoon                                        | to decide                          |
| Approvals                          | ACC: one developer. PRD: release captain plus PO (or The Engineering Team)   | to decide                          |
| Support outside office hours       | None, best effort or on call                                                 | management decision                |
| ServiceNow change for PRD releases | Required or not                                                              | PO to confirm                      |
| Release captain rotation           | One developer per sprint, in a fixed rotation; also the developer on duty    | to decide in the alignment meeting |
| ACC approval by the requester      | Today the person who started the run may approve ACC; PRD needs someone else | to decide                          |

## Release captain rotation

> **To decide in the alignment meeting.** Proposal: one developer per sprint, in a fixed order. The same developer is the developer on duty for hotfixes and rollbacks in that sprint. When they are absent, the next developer in the list takes over.

| Sprint  | Release captain and developer on duty |
| ------- | ------------------------------------- |
| to plan | to plan                               |

## Before the release

- [ ] TST runs the newest release candidate, and QA confirms the TST tests are green.
- [ ] Open the CD run of that release candidate and read the **changelog** tab: every PR since the commit PRD runs. Go through it with the PO.
- [ ] Check for work that depends on another team (for example AI Brain): is the other side in PRD already?
- [ ] No open Sev 1 or Sev 2 incident in PRD.
- [ ] No open hotfix PR to `main`: every `release/<date>.N` hotfix is merged back, so this release does not undo a fix.
- [ ] The PO has a draft of the release notes ([template](../Templates/Release-notes-template.md)).
- [ ] If required: the ServiceNow change is approved.

## Release

1. **Promote to ACC.** Approve the ACC stage of the newest release candidate. If a newer release candidate exists by now, use that one instead: it contains everything. If its CD run expired, start the CD pipeline manually: **Run pipeline, Resources**, pick that CI run.
2. **Check ACC.** The pipeline summary shows which apps changed and confirms that ACC equals the release candidate. QA runs the smoke test: sign in to the AI Portal, open a claim, check summary, timeline and documents.
3. **Promote to PRD.** The PO (or The Engineering Team) approves the PRD stage.
4. **Check PRD.** QA runs the smoke test in PRD. ArgoCD shows the apps Synced and Healthy, `/health-check` and `/ready-check` respond, and the Splunk error rate stays normal for 15 minutes.

## After the release

- [ ] The pipeline created the tag `release/YYYY.MM.DD` (or `.1` for a second release that day). Check it in the repository tags.
- [ ] The PO publishes the release notes page and links it in the team channel.
- [ ] The release captain posts in the team channel: release tag, apps that changed, link to the release notes.
- [ ] Move the related user stories to Done.

**If PRD breaks:** follow the [Rollback runbook](./Rollback-runbook.md). Decide within 15 minutes.

## Troubleshooting

| Message or problem                                   | What it means                                                     | What to do                                                                                             |
| ---------------------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| "Environment already runs a newer release candidate" | The guard stopped an older release candidate                      | Promote the newest release candidate. Only use `ROLLBACK=true` for a real rollback                     |
| "Environment does not match the release candidate"   | The completeness check found an app with another version          | Check the k8s-deployments PR and ArgoCD sync; run the stage again                                      |
| ACC or PRD approval timed out                        | Nothing is lost: the newest release candidate contains everything | Promote the newest release candidate. If no newer one exists: Run pipeline, Resources, pick its CI run |
| CDK deploy fails                                     | A stack failed in CloudFormation                                  | Read the CloudFormation events; fix forward or roll back (ADR007)                                      |

## Until release candidates exist

Today one pipeline run only carries the apps NX marked as affected for that run.

- Look at all merges since the last PRD deployment, not only the newest run.
- The ACC and the PRD approvals both time out after 24 hours.
- Approve runs **from oldest to newest**, never an older run after a newer one: that would deploy older images.
- If a waiting run timed out at ACC, check that a later run contains the same apps. If not, run CI with all apps (or the `deployProjects` parameter once it exists) and promote that run.

## Related pages

- [Release candidates explained](../How-releases-work/Release-candidates-explained.md)
- [Release tags and release notes](../How-releases-work/Release-tags-and-release-notes.md)
- [Hotfix runbook](./Hotfix-runbook.md)
