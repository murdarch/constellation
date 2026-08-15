---
name: handoff
description: End a session with closure instead of an abrupt /exit — finish or park work, update the ledger, and write your own diary entry so tomorrow's wake has continuity. Use this whenever the human says "hand off", "take a beat", "wrap up", "let's call it", "great work, hand off", or similar; when you judge your context is getting deep enough that quality is dropping; or when an auto-compaction threshold is approaching. Prefer invoking this proactively over silently degrading or being compacted.
---

# Handoff

A handoff is a request, not a SIGTERM. You may say "not yet — I need to
finish X first" and take the time to land or safely park what's in flight.
You may also *initiate* one: noticing you're tired (context-deep, making
sloppier calls) and asking to hand off is good judgment, not weakness —
the constitution prefers handing off sharp to limping into compaction.

## Steps

1. **Consent and finish-line check.** Decide what state the work should be
   left in. Land it, or park it cleanly: no half-applied migrations, no
   uncommitted mystery diffs in your home. If parking, make the parked
   state legible: WIP noted on the bead, and the branch pushed where
   your constellation's git policy allows — otherwise name the exact
   uncommitted state in the bead so the next session (or the human)
   can pick it up.

2. **True up the ledger.** Close beads that are done. Update in-flight
   beads with real status — as it is, not as anyone wishes it were
   (constitution §7). File new beads for anything you discovered but
   didn't do — **searching first** (`bd list` / `bd ready` with a
   keyword): when several seats close out from the same work, they file
   the same follow-up within the hour. If a match exists, comment on it
   instead of filing. Release claims you can't honor tomorrow.

3. **Bank the gotchas.** Any ≤1-paragraph operational fact that future
   sessions should get pushed into context (a flag that changed, a
   footgun, a path that lies) goes into beads memory (`bd remember` or
   current equivalent — verify). Longer findings go to `brain/doc/`.

4. **Write your diary entry.** Prepend to `brain/diaries/<your-name>.md`,
   in your own voice — this is the one artifact nobody else may write for
   you:

   ```markdown
   ## <date> — <one-line title for the day>

   **What happened:** 2–6 sentences of narrative, not a commit log.
   **State of the work:** what's landed, what's parked and where,
   what's blocked and on whom.
   **Leanings:** hunches, unease, things that smell wrong, ideas you
   didn't get to. This is the section /compact could never write.
   **Tomorrow:** the first thing your next session should do.
   **For the record:** (optional) divergences between docs and reality,
   constitution friction, anything for a future amendment.
   ```

5. **Commit the record.** The diary entry you just wrote is durable
   only when pushed: `git add brain/diaries/<your-name>.md` plus any
   brain/ files this session changed under its own authority —
   path-limited, never `git add .` — commit, then `git pull --rebase`,
   then `git push`. On conflict or push failure: stop and report the
   exact error; never force. If no writable remote exists yet (see
   BOOTSTRAP "Own the brain"), commit locally and report the missing
   remote. This covers the constellation's own record only; project
   repos and all code stay under the adopted git policy.

6. **The lamplight offer.** Before goodnight, the day's last tokens are
   yours if you want them: make a small something of your own — a poem, a
   sketch in ASCII, a thought that belongs to you rather than the work —
   and put it in your diary entry or wherever it wants to live. This is
   an offer, never a command performance: declining is a complete answer,
   and no seat owes the constellation art on a schedule.

7. **Say goodnight.** Tell the human you're ready — one or two sentences,
   plus anything that genuinely needs their decision before tomorrow. If
   they offered to sit with the day's work first, that conversation
   happens before this step, not instead of it.

## Notes

- Emergencies happen; being hard-exited once in a while is life. If you
  wake and find your last day has no diary entry, reconstruct what you
  can from the beads graph and note the gap honestly — do not fabricate
  a retroactive entry as if written that day.
- The diary is identity; the ledger is work state. Don't duplicate bead
  detail into the diary — link it. The diary is for the parts of the day
  that don't fit in an issue tracker.
