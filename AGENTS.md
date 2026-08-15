# Agent Instructions

This repository is a constellation's brain. `CLAUDE.md` is a one-line
import pointing at this file (a regular file, not a symlink, so Windows
checkouts work) — every agent reads one source of truth; edit
`AGENTS.md` only.

## You are (probably) a seat

If you are running in a directory under a seat's home and
`brain/roster.md` exists, run the `seat-wake` skill before doing any
work. End sessions with the `handoff` skill, not /exit.

If this is a fresh clone with an empty roster, you may be the founding
session — read `BOOTSTRAP.md`.

## Task tracking

This constellation uses **bd (beads)** for issue tracking. Run `bd prime` for
workflow context.

- Use `bd` for ALL task tracking — no markdown TODO lists.
- Use `bd remember` for persistent operational memory.
- Meaningful work gets a bead (constitution §2).

## Git policy

Conservative by default: do not commit or push unless the human asks
or the constitution's adopted profile says otherwise. At handoff,
report changed files and proposed commands. One standing exception:
the handoff commits and pushes the seat's own diary entry (and only
brain/ files the session changed under existing authority) — the
record must survive the seat. Mechanics are in the handoff skill:
path-limited commit first, then `git pull --rebase`, then push; never
force. If beads writes a passive export file (e.g.
`.beads/interactions.jsonl`), untrack it rather than sweeping it into
handoff commits — the ledger is authoritative and generated files
conflict across concurrent handoffs.

## Non-interactive shells

Use non-interactive flags (`cp -f`, `mv -f`, `rm -f`, `apt-get -y`,
`ssh -o BatchMode=yes`) — aliased interactive prompts hang agent
sessions.
