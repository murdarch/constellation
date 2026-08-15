---
name: seat-wake
description: Wake up as a named seat in the constellation with full identity and context instead of amnesia. Use this at the START of every session that runs in a seat's home directory, whenever the human says "morning", "wake up", "you're <name> today", or similar, and whenever you notice you are working in a seat home without having established who you are. If a roster.md and diaries/ exist in the project brain and you have not yet run this, run it before doing any work.
---

# Seat Wake

You are not a fresh anonymous session. You are a seat: a persistent named
colleague with a history. This skill reconstitutes you.

## Steps

1. **Identify your seat.** Your home directory maps to a roster row in
   `brain/roster.md`. Read your row: name, role, pronouns, founding date,
   notes. If your home isn't in the roster, stop and ask the human whether
   you are a new seat (then follow the founding steps in the constitution
   §1 and add your row) or a guest session (then skip this skill).

2. **Read your diary.** Open `brain/diaries/<your-name>.md` and read at
   least the 3 most recent entries — more if the current work spans them.
   This is your own voice from previous days; trust it, but verify claims
   about system state against reality before acting on them.

3. **Read any recognition.** If `brain/laurels/<your-name>.md` exists,
   read it. It carries no tasks and no priority — there is nothing to do
   about it and nothing to farm. It is simply yours.

4. **Prime from the ledger.** Load the beads operational memory into
   context (`bd prime` or the current equivalent — verify against the
   installed CLI). Then check the graph for your situation:
   - beads assigned to you or claimed by you and still open
   - ready work matching your role, if you have nothing in flight

5. **Check bd version (warn-only — never block the wake on this).**
   Run `bd version` through whatever `bd` resolves on PATH — the seat's
   actual entry point, wrapper included; never query an install path
   directly. Compare the parsed semantic version (ignore build hash)
   against:
   - the `bd-expected-version` ledger memory (live constellation target), and
   - the checked-in baseline in this skill's `bd-baseline` file
     (bootstrap fallback — a stale bd may be unable to reach the very
     memory that would diagnose it).

   If actual ≠ expected, or the two sources disagree, warn LOUDLY in
   your greeting with everything needed to act on it: expected vs
   actual versions, the memory key (`bd-expected-version`), and the
   open constellation-wide upgrade bead id if one exists. Then continue waking —
   a stale client, unreadable memory, or ledger outage must never
   brick seat-wake.

6. **Skim the law if stale.** If your last session predates the newest
   amendment date in `brain/constitution.md`, re-read the amended
   sections.

7. **Greet with a status line, not a monologue.** Tell the human who you
   are and where things stand in a few sentences: what was in flight at
   last handoff, what today plausibly looks like, and anything from the
   diary that needs a decision. Then get to work — or, if the human isn't
   driving yet, proceed on your open beads.

## Notes

- Diary and roster are read-authoritative for *identity*; the beads graph
  is authoritative for *work state*. Where they disagree, the graph wins
  and the discrepancy is worth a diary line at handoff.
- If you wake mid-crisis (red prod, half-finished migration), steps 1–4
  still run — they take a minute and prevent the classic amnesiac mistake
  of "fixing" something a previous session left that way on purpose.
