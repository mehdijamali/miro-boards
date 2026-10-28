# Miro board: Release Strategy Decision Meeting

Build guide with the exact content for every frame. Source: `docs/wiki/Release-and-delivery/` (ADR003–ADR007 and the team agreements in Runbooks/Release-to-PRD.md).

Build time: about 75 minutes. Tip: build the ADR003 frame completely, then duplicate it (⌘D) for ADR004–ADR007 and only change the text.

---

## Part 1. Miro actions you will use

| Action | How in Miro |
|---|---|
| New board | Dashboard → **+ Create new** → **Blank board**. Click the board name (top left) and type `Release Strategy Decision Meeting – 29 Oct 2026`. |
| Frame | Press **F** (or the Frame icon in the left toolbar). In the frame panel choose **16:9**, click on the canvas. Drag a corner while holding **Shift** to make it about twice as big (it keeps 16:9). Double-click the frame name (above the frame, top left) to rename it. |
| Sticky note | Press **N**, click inside the frame, type. Change colour in the small toolbar that appears above the selected sticky. |
| Text | Press **T**, click, type. Change size in the toolbar above it. |
| Shape | Press **S** → choose **Rectangle**, drag, then type inside. Fill colour and border in the toolbar above. |
| Arrow | Press **L**, drag from one object to another. |
| Image | Drag a PNG from Finder onto the frame. |
| Table | Left toolbar **+** (More tools) → search **Table**. Or paste a screenshot instead (faster). |
| Group | Select several objects → **⌘G**. |
| Lock | Select → right-click → **Lock** (⌘⇧L). Lock titles, proposals and option stickies so nobody moves them by accident. |
| Duplicate | Select the frame → **⌘D**. |
| Frame order | Left toolbar **Frames** icon (or bottom-left) → drag frames into order 0–9. **Present** in that panel gives slide mode. |
| Voting | Left toolbar **+** → search **Voting** → **Select items for voting** → click the vote shapes → set **Votes per person**, **Anonymous** on/off, **Duration** → **Start voting**. When it ends: **Results** → **Save results to board**. |
| Timer | Top-right collaboration toolbar → **Timer** (clock icon) → set minutes → Start. Everyone sees it. |
| Bring people along | Click your avatar (top right) → **Bring everyone to me**, or ask people to click your avatar to **follow** you. |
| Share | **Share** (top right) → add the team or copy the link with **Can edit**. |

Voting and Timer need a paid Miro plan (NN's plan normally has them). Fallback without Voting: each person makes one sticky with their name in their own colour and drags it onto the option they choose.

### Colour legend (use it everywhere)

| Colour | Meaning |
|---|---|
| Light gray sticky | An option that is not proposed |
| Green sticky + ⭐ | The proposed option |
| Yellow rectangle | The proposal |
| Pink / red sticky | Costs and negative consequences |
| Orange sticky | Concern or question from a participant |
| Violet sticky | Action point (the facilitator writes these) |
| Blue sticky | Information or evidence |

---

## Part 2. Layout of every ADR frame (frames 2–6)

```
┌──────────────────────────────────────────────────────────────────────┐
│ TITLE (Text, 72 pt, bold)                QUESTION (Text, 40 pt)        │
├──────────────────────┬──────────────────────┬────────────────────────┤
│ OPTIONS              │ PROPOSAL             │ 💬 CONCERNS             │
│ label (Text, 32 pt)  │ yellow rectangle     │ light orange rectangle  │
│ one sticky per       │ with bullet text     │ (empty, the team adds   │
│ option, stacked      │                      │ orange stickies here)   │
│ vertically           ├──────────────────────┼────────────────────────┤
│ proposed = green ⭐  │ COSTS                │ ✅ ACTION POINTS         │
│ others = gray        │ pink stickies        │ light violet rectangle  │
│                      │                      │ (empty, you fill it)    │
├──────────────────────┴──────────────────────┴────────────────────────┤
│ 🗳 VOTE   [✅ Accept]   [🟡 Accept with action points]   [❌ Reject]       │
│ three large rounded rectangles: green / yellow / red fill               │
└──────────────────────────────────────────────────────────────────────┘
```

How to build it once:

1. Frame (F), 16:9, enlarge ×2, name it as listed per frame below.
2. Text (T) top left: the title. Text top right: the question.
3. Three columns: draw three tall rectangles (S) with a thin gray border and no fill as column backgrounds. Put a text label on top of each: `OPTIONS`, `PROPOSAL`, `💬 CONCERNS`. Split the middle and right columns horizontally: the lower half of the middle column is `COSTS`, the lower half of the right column is `✅ ACTION POINTS`.
4. Fill the right-column areas: CONCERNS rectangle light orange, ACTION POINTS rectangle light violet. Inside CONCERNS add small gray text: `Add an orange sticky (N): one concern or question per sticky. Put your initials on it.`
5. Bottom row: text `🗳 VOTE` and three rounded rectangles (S → Rounded rectangle), each about the width of a column:
   - Green fill, text `✅ Accept`
   - Yellow fill, text `🟡 Accept with action points`
   - Red fill, text `❌ Reject`
6. Lock everything except the CONCERNS and ACTION POINTS areas (select all items → right-click → Lock; then unlock the two empty rectangles, or simply leave stickies free to be added on top).
7. Duplicate the frame 4 times (⌘D) and change the text.

---

## Part 3. Exact content per frame

### Frame 0 · Welcome and how we vote

Frame name: `0 · Welcome`

Layout: title at the top, then three columns (Goal and agenda | How we vote | Colour legend), voters at the bottom.

**Title (Text, 80 pt):**
```
Release Strategy Decision Meeting
```
**Subtitle (Text, 36 pt):**
```
BIC Claim Assistant · 29 October 2026 · Decide ADR003–ADR007 and the release team agreements
```

**Left column – yellow rectangle "Goal":**
```
🎯 Goal
Move 5 proposed ADRs to accepted or rejected, and agree how we work around a release.

Agenda (90 min)
0:00  Why we are here (frames 0–1)
0:10  ADR003 How do we release?            20 min
0:30  ADR004 Tags, changelog, release notes 10 min
0:40  ADR005 Preview environment            10 min
0:50  ADR006 Hotfix process                 12 min
1:02  ADR007 Rollback                       10 min
1:12  Team agreements                       10 min
1:22  Results, owners, next steps            8 min
```

**Middle column – blue rectangle "How we vote":**
```
🗳 How we vote (per ADR)
1. 2 min: the owner presents the proposal
2. 3 min: silent writing – orange stickies in 💬 CONCERNS
3. Discussion – the owner turns concerns into ✅ ACTION POINTS
4. 1 min: vote with Miro Voting – 1 vote per decider
   ✅ Accept   🟡 Accept with action points   ❌ Reject
5. Passed = most deciders accept (✅ + 🟡) and nobody
   has a blocking reason. Blocking reasons are written down.

An accepted ADR does not change anymore; a new ADR supersedes it.
Out of time? Result becomes 🟡 with action points. We move on.
```

**Right column – colour legend.** Make one small sticky of each colour with this text:
- Gray: `Option (not proposed)`
- Green: `⭐ Proposed option`
- Yellow: `Proposal`
- Pink: `Cost / negative consequence`
- Orange: `Your concern or question`
- Violet: `Action point`
- Blue: `Evidence / info`

**Bottom – "Deciders" (Text 32 pt) and one sticky per person:**
```
Deciders: BIC team – Engineers and PO
```
One sticky per decider with their name (give every person their own colour if you use the name-sticky fallback). Add one more sticky: `Advisor: QA`.

**Links (Text, small, bottom right):**
```
📖 Wiki: Release and delivery → Decisions (ADR003–ADR007)
📖 Pre-read: How releases work → Release candidates explained
🎞 Presentation: docs/rfc/2026-10-15-release-strategy-presentation.html
```
(Paste the real wiki URLs once the wiki is published.)

---

### Frame 1 · Why we are here

Frame name: `1 · The problem`

**Title:** `Why we need to decide`

**Top half – four blue stickies (large size), side by side:**
1. ```
   23 of 39
   pipeline runs from main timed out at the ACC approval
   (mid-June – 15 Oct 2026).
   Their changes only reached PRD if a later run rebuilt the same apps.
   ```
2. ```
   31 of 72
   CD runs deployed a feature branch into the shared TST.
   A fix/ branch reached ACC. A deploy/ branch reached PRD –
   its commit never reached main.
   ```
3. ```
   0 release tags
   No changelog, no release notes.
   k8s-deployments records the wrong commit:
   nobody can say quickly what runs in PRD.
   ```
4. ```
   No agreed hotfix or rollback process
   Approving an older run silently deploys older images.
   The ArgoCD rollback button does not stick.
   ```

**Bottom half – the target flow.** Draw six rounded rectangles in a row and connect them with arrows (L):

`Pull request` → `main` → `Release candidate (RC)` → `TST` → `ACC` → `PRD`

Under the RC box, a small yellow sticky:
```
RC = the version of every app + CDK stacks + WSO2 config,
built from one commit of main. Build once, promote everywhere.
```
Under PRD, a small yellow sticky:
```
Tag release/YYYY.MM.DD + changelog + release notes
```
Above TST, a small green sticky:
```
Preview environment for feature branches (TST stays = main)
```

**Bottom right – text:**
```
Today we decide 5 things:
ADR003 How we release · ADR004 How we mark releases
ADR005 Where we test branches · ADR006 Hotfixes · ADR007 Rollback
```

---

### Frame 2 · ADR003 How do we release?

Frame name: `2 · ADR003 Release candidates`

**Title:** `ADR003 · Promote release candidates to PRD`
**Question:** `How do we release – so nothing merged is missed, nothing goes back by accident, and we know what is in PRD?`

**OPTIONS (stack vertically):**

Gray – `A. Keep the current process`
```
A. Keep the current process
Approve the right runs, re-run what was missed.
✗ Changes still missed  ✗ Old approvals can roll PRD back
```
Gray – B
```
B. Since the last release tag
Tag every PRD release; CI builds everything changed since that tag (Score team does this).
✓ Small, proven  ✗ Old approvals can still deploy older images  ✗ Rollback = rebuild
```
Green ⭐ – C
```
⭐ C. Release candidates
Every merge to main publishes an RC: version of every app + CDK + WSO2. Promote TST → ACC → PRD.
✓ Meets all 5 decision drivers
```
Gray – D
```
D. Release branches
✗ Branch overhead and merge-back mistakes
✗ Conflicts with trunk-based development and NX affected
```
Gray – E
```
E. Always build and deploy everything
✗ Every release restarts every app
✗ PRD gets images ACC did not test
```
Gray – F
```
F. A pipeline per app
✗ 5 apps × 3 environments = 15 approvals
✗ Portal + BFF and ingestion + CDK need manual coordination
```

Optional: below the options, a screenshot of the comparison table from ADR003 (open the ADR in VS Code Markdown preview, ⌘⇧4 on macOS, drag the PNG in). Or build it with the Table tool:

| | A | B | C | D | E | F |
|---|---|---|---|---|---|---|
| No missed changes | ✗ | ✓ | ✓ | ✓ | ✓ | ~ |
| No accidental rollback | ✗ | ~ | ✓ | ✓ | ~ | ~ |
| Know what is in PRD | ✗ | ✓ | ✓ | ✓ | ✓ | ~ |
| Fast rollback | ✗ | ~ | ✓ | ~ | ~ | ✓ |
| Same image in ACC and PRD | ~ | ~ | ✓ | ~ | ✗ | ✓ |
| Only changed apps restart | ~ | ~ | ✓ | ✓ | ✗ | ✓ |
| Effort | none | S | M | M | S | L |
| Daily overhead | high | low | low | high | low | high |

**PROPOSAL (yellow rectangle):**
```
Proposal: C. Release candidates
• An RC = the version of every app + CDK stacks + WSO2 config, from one commit of main
• CD compares the RC with the environment and only updates what differs
• A guard refuses an older RC, unless it is a deliberate rollback (ROLLBACK=true)
• Build once: the image tested in ACC is the image in PRD
• Rollback = promote the previous RC (see ADR007)
• Interim if we want the smallest first step: B (its release tag step is reused by C)

📏 Rule that comes with it:
Only merge to main what may go to PRD with the next release.
Unfinished work stays in its PR (show it in the preview environment).
```

**COSTS (pink stickies):**
- `Pipeline work: RC manifest in CI, comparison + guard in CD, all CDK stacks and WSO2 per RC`
- `Keep the last 5 PRD RCs: pipeline runs, images, artifacts`
- `New concept for the team`
- `main must stay releasable – we cannot leave one merged change out`
- `To check: republishing unchanged WSO2 config is harmless; cdk deploy --all stays fast enough (incl. cross-account Felet Lambda)`

**💬 CONCERNS / ✅ ACTION POINTS:** empty.

**🗳 VOTE:** the three vote shapes. **Extra sub-vote** to the right of them, separated by a vertical line, with the label `Sub-vote: how do we start?`:
- Blue rounded rectangle `🚀 Go straight to C`
- Blue rounded rectangle `🪜 B first as interim, then C`

---

### Frame 3 · ADR004 Tags, changelog and release notes

Frame name: `3 · ADR004 Tags and changelog`

**Title:** `ADR004 · Tag PRD releases and generate the changelog`
**Question:** `How do we mark a PRD release, and how do approvers and users learn what is in it?`

**OPTIONS – two groups with a small text label above each.**

Label `Release marker`:
- Green ⭐ `⭐ M1. Date tag (CalVer)\nrelease/2026.10.20 – hotfix release/2026.10.20.1\n✓ Everyone understands "the release of 20 October"  ✓ Easy to automate`
- Gray `M2. Semantic version (nx release) – v1.4.0\n✗ Needs strict commit discipline + release commit  ✗ "Major" is hard to define for a UI`
- Gray `M3. Build id only (549747)\n✗ Identifies one pipeline run, not a release`
- Gray `M4. No marker, pipeline history\n✗ Invisible in Git, disappears with run retention`

Label `Changelog and release notes`:
- Gray `C1. CHANGELOG.md in the repo\n✗ Pipeline must push to main (policy bypass) or a release PR`
- Green ⭐ `⭐ C2. Changelog in the pipeline, before approval\n✓ Approvers see what they approve  ✓ Pipeline does not write to main`
- Green ⭐ `⭐ C3. Release notes on the wiki, by the PO\n✓ Written for people, can link ServiceNow change, tests, stories`
- Green ⭐ (smaller) `⭐ C4. Draft from linked work items (helper for C3)\nOnly as good as PR ↔ work item linking`
- Gray `C5. A change file per PR (changesets)\n✗ Friction for every PR, little gain at our size`

(Where you see `\n`, press Enter for a new line.)

**PROPOSAL (yellow rectangle):**
```
Proposal: M1 + C2 + C3 (C4 pre-fills the notes)
• Annotated tag release/YYYY.MM.DD[.N], placed by the pipeline after a successful PRD deployment
• Tags are never moved or deleted (pipeline has "Create tag" only, no force push)
• Changelog with nx release changelog --dry-run, shown in the CD run before ACC approval,
  starting from the commit PRD runs (not simply the newest tag – matters after a rollback)
• The PO writes short release notes on the wiki per PRD release (template ready)
• PR titles must follow  type(#ticket): description  – checked by the PR pipeline
• First tag: release/2026.09.30 on 384f0878 (what PRD runs today)
```

**Blue evidence sticky (near the proposal):**
```
Since 15 April 2026: 78 PRs merged, 60 (77%) follow Conventional Commits.
The biggest features do not, e.g. "Feature: Implement Automatic Summary 2.0".
```

**COSTS (pink stickies):**
- `PR title check = a new rule for every PR`
- `Release notes = a new task for the PO every release`
- `Changelog is not a file in the repo (C1 can be added later)`

**VOTE:** three vote shapes.

---

### Frame 4 · ADR005 Preview environment

Frame name: `4 · ADR005 Preview environment`

**Title:** `ADR005 · Test feature branches in a preview environment`
**Question:** `Where do developers, UX and the PO try unfinished work, while TST, ACC and PRD only get main?`

**Blue evidence sticky at the top of OPTIONS:**
```
31 of 72 CD runs put a feature branch in TST.
A fix/ branch reached ACC; a deploy/ branch reached PRD.
```

**OPTIONS:**
- Gray `E1. TST etiquette\nBook TST in the channel, deploy main back afterwards\n✓ Free  ✗ People forget – TST still differs from main`
- Green ⭐ `⭐ E2. Preview environment in TST\nbic-claim-assistant-web-preview + bic-claim-assistant-api-preview, deployed from any branch by a manual pipeline (same as bic-portal-preview)\n✓ TST stays = main  ✓ Stable address for UX and PO  ✓ No Cloud Platform Team review needed`
- Gray `E3. New DEV environment\n✓ Full isolation  ✗ Needs Cloud Platform Team, AWS, secrets, WSO2  ✗ AI Brain / FELET may have no DEV`
- Gray `E4. Environment per PR\n✗ Lots of Cloud Platform Team work, wildcard DNS + TLS  ✗ Azure AD discourages wildcard redirect URIs`

**PROPOSAL (yellow rectangle):**
```
Proposal: E2. Preview environment in TST
Together with it:
• Branch control on ACC and PRD: only main and hotfix/*
• The CD pipeline starts automatically only for main (and hotfix/*)
• TST is always main → acceptance in TST means something
• We revisit DEV or per-PR environments if one preview is not enough
```

**COSTS (pink stickies):**
- `One branch at a time – the team agrees who uses it`
- `Portal + BFF only: CDK and Lambda changes still share TST`
- `Setup: preview pipeline, 2 apps in k8s-deployments, Azure AD redirect URI + API_BASE_URL`

**VOTE:** three vote shapes.

---

### Frame 5 · ADR006 Hotfix process

Frame name: `5 · ADR006 Hotfix process`

This frame has more content. Make it slightly bigger, or put the severity table across the top of the middle + right columns.

**Title:** `ADR006 · Handle PRD incidents with a hotfix process`
**Question:** `When is something a hotfix, and how do we get a fix to PRD fast without skipping safety or losing the fix?`

**OPTIONS:**
- Green ⭐ `⭐ H1. Fix forward on main (default)\nUrgent PR → new RC → promoted quickly\n✓ Normal path, no merge back  ✗ Releases everything else on main too`
- Green ⭐ `⭐ H2. Hotfix branch from the release tag (when main is not releasable)\nhotfix/<ticket> → RC → ACC → PRD → tag release/<date>.1 → merge back the same day (Score team does this)\n✓ Only the fix goes to PRD  ✗ Needs merge-back discipline`
- Gray `H3. Cherry-pick into a release branch\n✗ We have no release branches (ADR003 option D)`
- Green ⭐ `⭐ H4. Configuration only\nSecrets via Key Vault; envVars, CPU, memory via an emergency/ branch in k8s-deployments\n✓ Fastest, no build  ✗ Config only, not code`

**Severity table (Table tool, 3 rows, or three stickies in a row: red, orange, gray):**

| Severity | Examples | Response |
|---|---|---|
| 🔴 Sev 1 | AI Portal down or claims cannot be opened; data of the wrong claim shown; personal data exposed; actively exploitable vulnerability | Hotfix now. Roll back first if the last release caused it. |
| 🟠 Sev 2 | Summary or timeline not loading for many users; event ingestion stopped; critical vulnerability with exposure | Hotfix within the business day |
| ⚪ Sev 3–4 | Cosmetic issues, bugs with a workaround | Normal fix in the next release |

**PROPOSAL (yellow rectangle):**
```
Proposal: H1 by default · H2 when main is not releasable · H4 for configuration only

Rules
• Last release < 48 h ago caused it and rollback restores service? → Roll back first (ADR007), then fix calmly
• Never skip ACC. At least 1 reviewer who is not the author
• The Engineering Team declares the hotfix and severity; the PO approves PRD
• Post in the team channel at start and end
• Postmortem within 5 working days for Sev 1 and Sev 2
• H2: open the merge-back PR to main at the start; complete it the same day with a normal merge (no squash)
• No PRD release while a hotfix PR to main is still open
```

**COSTS (pink stickies):**
- `CI and CD must accept hotfix/* branches; branch control must allow them`
- `Branch policy on main must allow a normal merge (no squash) for hotfix/* PRs`
- `Until CI uses the release tag as NX base, a hotfix must be one commit`
- `main must stay releasable for H1`

**Gray sticky in the bottom corner, label "Not in this ADR":**
```
Support outside office hours → management decision
```

**VOTE:** three vote shapes.

---

### Frame 6 · ADR007 Rollback

Frame name: `6 · ADR007 Rollback`

**Title:** `ADR007 · Roll back by redeploying the previous release candidate`
**Question:** `How do we go back to the previous release quickly, safely and visibly?`

**OPTIONS:**
- Gray `R1. Feature flag off\n✓ Seconds, one feature only  ✗ We have no flags yet (US 441556) – does not replace rollback`
- Green ⭐ `⭐ R2. Redeploy the previous RC\nCD run for the previous RC with ROLLBACK=true\n✓ Minutes, no rebuild  ✓ Apps + CDK + WSO2  ✓ Visible in history`
- Green ⭐ (smaller, label "interim") `⭐ R3. Rebuild from the previous release tag (until RCs exist)\n✓ Works today  ✗ Slow: build + 3 environments  ✗ New bits  ✗ Needs quick win N5 (deployProjects)`
- Gray `R4. ArgoCD rollback button\n✗ ArgoCD reapplies Git immediately – does not stick  ✗ No audit trail`
- Gray `R5. Canary / blue-green (Argo Rollouts)\n✓ Automatic  ✗ Not installed on the Cloud Platform Team clusters – future option`

**PROPOSAL (yellow rectangle):**
```
Proposal: R2 by default · R3 until RCs exist · never R4 · no manual k8s-deployments changes

⚡ Trigger: Sev 1 within 48 h after a release · 5xx above 2% for 10 min in Splunk ·
   failing health or smoke checks · security exposure
👤 Decision: release captain or The Engineering Team, within 15 minutes; PO informed.
   Rollback stays a manual decision.
🗄 Retention: keep the last 5 PRD RCs (runs, images, CDK + WSO2 artifacts).
   RCs that reached ACC get a 30-day retention lease.
🔁 Rehearsal: a rollback drill in ACC every quarter.
```

**COSTS (pink stickies):**
- `Needs release candidates first; until then R3 + quick win N5`
- `Secrets and values.json (envVars, CPU, memory) are not part of a release – revert them separately`
- `Artifactory / ECR lifecycle must keep at least 5 PRD versions (to confirm)`
- `A broken event ingestion version may need an event replay (US 390288)`
- `Replaced or deleted CDK resources cannot be restored – review the CDK diff in PRs`

**VOTE:** three vote shapes.

---

### Frame 7 · Team agreements

Frame name: `7 · Team agreements`

**Title:** `Team agreements – not ADRs, quick polls`
**Subtitle:** `One vote per question. 5 votes in total. Unhappy with all options? Add an orange sticky next to the question.`

Layout: five horizontal rows. Each row has a question on the left (Text, 32 pt, bold) and 2–3 rounded rectangles (light blue) as answers. Separate rows with a thin line.

| # | Question (Text) | Answer shapes |
|---|---|---|
| 1 | `How often do we release to PRD?` | `When valuable, at least once per sprint` · `A fixed day every sprint` · `Every week` |
| 2 | `Release window?` | `Office hours, not on Friday afternoon` · `Office hours, any day` |
| 3 | `Approvals?` | `ACC: 1 developer · PRD: release captain + PO (or The Engineering Team)` · `Something else (add a sticky)` |
| 4 | `Release captain?` | `1 developer per sprint, fixed rotation, also developer on duty` · `Something else (add a sticky)` |
| 5 | `May the person who started the run approve ACC?` | `Yes – PRD always needs someone else` · `No – ACC needs someone else too` |

**Right side – rotation table (Table tool, 2 columns, 6 rows), title `Release captain rotation`:**

| Sprint | Release captain and developer on duty |
|---|---|
| Sprint __ | |
| Sprint __ | |
| Sprint __ | |
| Sprint __ | |
| Sprint __ | |

**Bottom – gray rectangle labelled `🚫 Not ours to decide today`, with two gray stickies:**
- `ServiceNow change for PRD releases: required or not? → PO confirms by ____`
- `Support outside office hours (none / best effort / on call) → management decision`

**Voting setup for this frame:** one voting session, select all 11 answer shapes, **Votes per person: 5**, **Max votes per object: 1**, duration 2 minutes. Say out loud: "one vote per question".

---

### Frame 8 · Parking lot

Frame name: `8 · Parking lot`

**Title:** `🅿️ Parking lot`
**Subtitle:** `Good questions that do not block a decision today. Each gets an owner before we close.`

Three columns (tall rectangles, light gray) with labels:
- `Needs another team` – pre-fill with blue stickies:
  - `Cloud Platform Team: image lifecycle in Artifactory / ECR keeps ≥ 5 PRD versions?`
  - `Azure AD: redirect URI for the preview address`
- `Needs more info` – empty
- `Later / nice to have` – pre-fill with gray stickies:
  - `Feature flags (US 441556) – separate session`
  - `Canary / blue-green with Argo Rollouts`
  - `CHANGELOG.md in the repository (ADR004 C1)`

---

### Frame 9 · Results and next steps

Frame name: `9 · Results`

**Title:** `Results – 29 October 2026`

**Table 1 (Table tool, 6 columns × 6 rows), label `Decisions`:**

| ADR | Result (✅ / 🟡 / ❌) | Votes ✅ / 🟡 / ❌ | Action points | Owner | Due |
|---|---|---|---|---|---|
| ADR003 Release candidates | | | | | |
| ADR004 Tags and changelog | | | | | |
| ADR005 Preview environment | | | | | |
| ADR006 Hotfix process | | | | | |
| ADR007 Rollback | | | | | |

**Table 2, label `Team agreements`:**

| Topic | Agreed |
|---|---|
| Release frequency | |
| Release window | |
| Approvals | |
| Release captain rotation | |
| Requester may approve ACC | |

**Right side – "Next work items" (violet stickies, fill in owner during the meeting):**
- `Build the release candidate pipeline (runbook: Set up release candidates) – owner: __`
- `Quick win N5: deployProjects parameter (needed for R3 rollback) – owner: __`
- `Branch control on ACC + PRD: only main and hotfix/*; CD auto-start only for main – owner: __`
- `Preview environment: pipeline + 2 apps in k8s-deployments + Azure AD redirect URI – owner: __`
- `PR title check type(#ticket): description in the PR pipeline – owner: __`
- `Create first tag release/2026.09.30 on 384f0878 – owner: __`
- `Retention leases + image lifecycle (keep last 5 PRD RCs) – owner: __`
- `CI/CD accept hotfix/* + branch policy normal merge for hotfix/* – owner: __`
- `Plan the first rollback drill in ACC – owner: __`
- `Update ADR status and date on the wiki – owner: __`

---

## Part 4. Running the meeting

### Today (after building)

1. Order the frames 0–9 in the Frames panel.
2. Lock titles, proposals, options and vote shapes.
3. Share with **Can edit** and send this message in the team channel:
   ```
   Tomorrow we decide our release strategy (ADR003–ADR007).
   Miro board: <link>
   Before the meeting: read "Release candidates explained" on the wiki (10 min)
   and put your concerns as orange stickies in 💬 CONCERNS on each ADR frame.
   ```

### Per ADR during the meeting (repeat for frames 2–6)

1. Click your avatar → **Bring everyone to me** on the frame.
2. Timer 2 min – present the proposal (yellow box), point at the ⭐ option.
3. Timer 3 min – silent writing in 💬 CONCERNS.
4. Group duplicate concerns (drag them together). Discuss. Write the result as violet stickies in ✅ ACTION POINTS.
5. **Voting:** + → Voting → select the 3 vote shapes (ADR003: also the 2 sub-vote shapes, and set Votes per person to 2, max 1 per object) → Votes per person 1 → Duration 1 min → Start.
6. **Results → Save results to board**, drag the result next to the vote shapes.
7. Write the result in Frame 9 right away.

### Closing (frame 9)

- Read out each result and owner.
- Every parking-lot sticky gets an owner or is moved to "Later".
- After the meeting: set each ADR status (accepted / rejected) and date on the wiki.
