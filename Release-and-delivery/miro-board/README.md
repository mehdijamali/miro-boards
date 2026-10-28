# Release Strategy Decision Meeting – Miro board

`release-strategy-board.svg` is the complete board for the 30-minute ADR vote on 29 October 2026, frames 0–9, in the Canvas Composer SVG format that the Miro connector reads and writes. It is the importable copy of the board, used instead of a Miro `.rtb` backup because the account the board was built in is on Miro's Free plan, and board backups need a paid plan.

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

## Recreate it in another Miro account

1. In the target Miro account, create an empty board.
2. In Claude, connect the Miro connector, signed in with that account. The organisation may need to allow the connector first.
3. Give Claude this file and the board link, with this prompt:

   > Recreate this Miro board on <board link>. Load the Canvas Composer skill first (canvas_get_canvas_composer_skill, then the tables format skill), then send `release-strategy-board.svg` unchanged with canvas_create_from_svg. Afterwards check the reported dimensions: the three tables should be about 1353 px (ADR003 options), 1132 px (rotation, team agreements) and 1582 px (decisions) wide; set data-scale if they come out smaller.

The file is about 51 KB, well under the connector's 200 KB limit, so it goes in one call. If a call fails, send it frame by frame: each `<g data-frame="…">` block is self-contained, wrapped in its own `<svg xmlns="http://www.w3.org/2000/svg">…</svg>`.

## Before the meeting

- Lock titles, proposals, options and vote shapes (select → right-click → Lock).
- Replace the wiki lines on frame 0 with the real wiki links.
- Share the board with the deciders with **Can edit**, so they can add concerns beforehand.
- Voting and the timer need a paid Miro plan. Start voting per ADR with **1 vote per person**: with 8 deciders an ADR passes with at least 5 votes for ✅ or 🟡 and no blocking reason.
- Once the board lives in a paid account, *Download board backup* from the board's menu on the dashboard gives an `.rtb` file that can be stored next to this one.
