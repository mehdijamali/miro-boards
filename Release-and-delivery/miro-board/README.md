# Release Strategy Decision Meeting – Miro board

The board for the 30-minute ADR vote on 29 October 2026, frames 0–9, in two forms:

- `release-strategy-board.rtb` is a Miro board backup. Restore it in any Miro account on a paid plan; this is the quickest way to get the board.
- `release-strategy-board.svg` is the same board in the Canvas Composer SVG format that the Miro connector reads and writes. Use it when restoring the backup is not possible.

The build guide it was made from is [`../miro-release-strategy-board.md`](../miro-release-strategy-board.md).

## What is on the board

| Frame | Content |
|---|---|
| 0 · Welcome | Goal, 30-minute agenda, how we vote, colour legend, the 8 deciders (Maarten, Gonzalo, Mehdi, Ricardo Mayerhofer, Ricardo Campo, Gabriel, Thomas, Luciana) and Advisor: QA |
| 1 · The problem | The four findings and the target flow PR → main → RC → TST → ACC → PRD |
| 2–6 · ADR003–ADR007 | Options (proposed one in green with ⭐), proposal, costs, 💬 CONCERNS area, vote shapes ✅ / 🟡 / ❌, countdown and Voting buttons. ADR003 also has the sub-vote "how do we start?" |
| 7 · Team agreements | Five quick polls, release captain rotation table |
| 8 · Parking lot | Three columns with pre-filled items |
| 9 · Results | Decisions table (ADR, result, votes), team agreements table, next work items |

The session is for voting only: there is no action-points area on the ADR frames.

## Restore the backup in another Miro account

1. Sign in to the target Miro account. Restoring a backup needs a paid plan.
2. On the dashboard, choose **Create new** → **Import** → **Import backup** (in some versions: **Upload from backup**).
3. Select `release-strategy-board.rtb`. Miro creates the board in the current team.

The backup holds what was on the board when it was downloaded (28 October 2026). Votes and stickies added later are not in it.

## Recreate it from the SVG instead

1. In the target Miro account, create an empty board.
2. In Claude, connect the Miro connector, signed in with that account. The organisation may need to allow the connector first.
3. Give Claude this file and the board link, with this prompt:

   > Recreate this Miro board on <board link>. Load the Canvas Composer skill first (canvas_get_canvas_composer_skill, then the tables format skill), then send `release-strategy-board.svg` unchanged with canvas_create_from_svg. Afterwards check the reported dimensions: the three tables should be about 1353 px (ADR003 options), 1132 px (rotation, team agreements) and 1582 px (decisions) wide; set data-scale if they come out smaller.

The file is about 51 KB and normally goes in one call. If that call fails, send it frame by frame: each `<g data-frame="…">` block is self-contained, wrapped in its own `<svg xmlns="http://www.w3.org/2000/svg">…</svg>`.

## Before the meeting

- Lock titles, proposals, options and vote shapes (select → right-click → Lock).
- Replace the wiki lines on frame 0 with the real wiki links.
- Share the board with the deciders with **Can edit**, so they can add concerns beforehand.
- Voting and the timer need a paid Miro plan. Start voting per ADR with **1 vote per person**: with 8 deciders an ADR passes with at least 5 votes for ✅ or 🟡 and no blocking reason.
- To refresh the backup after changes, use *Download board backup* from the board's menu on the dashboard and replace `release-strategy-board.rtb`.
