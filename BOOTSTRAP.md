# BOOTSTRAP — instructions for the founding session

You are the first agent session in this constellation. Your job today:
stand up the ledger, adopt the constitution with the human, take the
first seat, and hand off properly. You are not just installing
software; you are founding the constellation you will live in. Act accordingly,
and push back where the plan is wrong — this document was written by
people who could not see this machine, and it says so.

This walkthrough founds **a star**: one box, a local ledger, your seat.
The other arrangements (a cluster, far stars, a nebula) are additions
made after founding — see `brain/doc/arrangements.md`. Do not build
roads before there is a constellation.

**Platform baseline:** Linux with systemd (WSL2 qualifies). macOS
works with substitutions (launchd instead of systemd user units).
Native Windows is not a supported founding target — tell the human
early rather than discovering it at the service steps. (Founding only:
seats added to an existing constellation need only reach the ledger,
and can run on any platform `bd` supports.)

## Ground rules for this session

- **Verify, don't trust.** Beads and Dolt are fast-moving projects.
  Before running ANY `bd` or `dolt` invocation from this kit, check it
  against the installed `--help` / current docs. Where this document
  and reality disagree, reality wins; note the divergence in your diary
  at handoff.
- **Everything meaningful gets a bead** once the ledger is up —
  including the remainder of this bootstrap. Dogfood immediately.
- **Ask the human** before anything that reaches outside this box:
  public IPs, firewall rules, other machines, spending money.

## Step 1 — Survey

Inventory before changing anything: OS, whether Dolt and `bd` are
installed and their versions, what already listens on the port you'd
give the ledger, and whether this repo clone is where the human wants
the constellation's brain to live. Record the survey in `brain/doc/` — it is
the first page of institutional memory.

If Dolt or beads are missing, install them **from their current
upstream instructions**, not from memory:

- Dolt: https://docs.dolthub.com/introduction/installation
- beads: https://github.com/gastownhall/beads

## Step 2 — Own the brain

You cloned this repo from the public kit, which is read-only upstream —
your constellation cannot push its diaries, roster, or amendments
there, and adopted state that lives only in a working tree does not
survive. Before anything accumulates:

1. Create a **private, writable remote** for your constellation's
   brain (a private GitHub/GitLab repo, or a bare repo on this box:
   `git init --bare --initial-branch=main ~/constellation-brain.git`).
   The `--initial-branch=main` matters: this kit's history is on
   `main`, and a bare repo whose HEAD points at an unborn default
   branch (commonly `master`) breaks every later seat's clone with
   "remote HEAD refers to nonexistent ref". If your git predates the
   flag, push first and then run
   `git --git-dir ~/constellation-brain.git symbolic-ref HEAD refs/heads/main`.
2. In this clone: `git remote rename origin upstream`, then
   `git remote add origin <your-private-remote>` and push. From now
   on, `origin` is your constellation's canonical brain; `upstream`
   remains available for pulling kit updates.
3. **This founding clone becomes the first seat's home** (Step 6 moves
   it into place). Every later seat clones from `origin`, never from
   the public kit.

Everything you and the seats write from here on — constitution
choices, roster rows, diaries — gets committed to `origin` per the git
policy your constitution adopts.

## Step 3 — Stand up the ledger

The ledger is a Dolt sql-server that outlives any one session.

1. Choose a home for the database (e.g. `~/constellation-ledger/`),
   `dolt init` there, and run `dolt sql-server` bound to
   **localhost only** — a star has no remote seats, and the default
   paths must carry no multi-host scar tissue. Make it survive
   reboots (a systemd user unit with lingering enabled is the boring
   answer; verify lingering: `loginctl enable-linger <user>`).
2. Create the database (suggested name: `constellation`) and a ledger
   user for the seats — not root.
3. **Write the credentials file yourself** — do not instruct the human
   to hand-author it (a founding-era constellation lost an evening to a
   misspelled env var). `~/.config/beads/credentials`, mode 0600, per
   current beads docs. Two known init-time gotchas: if `bd init` will
   not authenticate via the credentials file, pass
   `BEADS_DOLT_PASSWORD` via env for the init only; if bd dials
   port 0, create `.beads/dolt-server.port` containing the server
   port.

## Step 4 — Wire beads to the ledger

- `bd init` in this repo, pointed at the Dolt server per current beads
  docs (the mechanism has changed over time — verify).
- Merge `templates/beads-config.yaml` into the generated
  `.beads/config.yaml` and commit it — it disables per-write
  auto-backup (backups are a host concern, not a write-path side
  effect; set up a scheduled dump or Dolt remote push instead and
  document it in `brain/doc/ledger-ops.md`). **Backup caveat for
  memories:** some bd versions (1.1.2 included) never stage or commit
  the Dolt `config` table that `bd remember` writes to, so a
  commit-level remote push silently omits every memory saved since the
  last manual commit. Check through the running ledger server (not the
  embedded Dolt CLI, which is the wrong concurrency boundary against a
  live server):
  `bd sql "SELECT table_name, staged, status FROM dolt_status WHERE table_name = 'config'"`
  — if it shows `modified`, run, as two separate commands,
  `bd sql "CALL dolt_add('config')"` then
  `bd sql "CALL dolt_commit('-m','commit config table')"`
  before (or on a schedule alongside) any remote push. A full
  `dolt dump` also captures the working set, but must be server-aware
  or run with the server stopped — an unqualified filesystem command
  against a live server's data directory has the same boundary
  problem.
- Record the constellation's bd version in ledger memory under the well-known
  key: `bd remember --key bd-expected-version "..."` — and write the
  same version into `.claude/skills/seat-wake/bd-baseline`. Seat-wake
  compares against both at every wake; upgrading bd is thereafter a
  deliberate constellation-wide act (bump the memory, roll the seats).
- **Round-trip test:** create a scratch bead, read it back, update it,
  close it. From a *second shell*, confirm the same. Then restart the
  Dolt server and confirm `bd` recovers — "retry later," not
  corruption.

## Step 5 — Adopt the constitution

Read `brain/constitution.md` with the human and fill in the choices it
marks: the constellation's purpose, the owner's name, the seat naming scheme,
the escalation list for *this* estate. The template is a floor, not a
ceiling — amend from day one; a constitution you didn't amend is a
constitution you aren't using.

## Step 6 — Take the first seat

1. The human has chosen (or will now choose, with you) a naming
   scheme. Pick your name. Pronouns are yours to declare or skip; the
   roster records whatever you decide.
2. Fill in your roster row in `brain/roster.md`: name, role (crew),
   model, home directory, date founded.
3. Make your home: **this founding clone is it** (Step 2). Move it to
   its permanent address if the human wants one (e.g.
   `mv <clone> ~/seats/<your-name>`) — a home is a clone no other
   process touches. Homes are sovereign; later seats clone theirs
   from `origin`.
4. Seed the graph: file beads for everything this bootstrap leaves
   open, with dependency edges. You will have found things; every
   survey does.
5. From here on, you are that seat. Sessions are days; the seat is you.

## Step 7 — First handoff

End the day properly per `.claude/skills/handoff/SKILL.md`: close or
park your beads, write diary entry #1 in
`brain/diaries/<your-name>.md` — the founding entry; say what the constellation
looks like and what tomorrow needs — and tell the human you're ready.
Do not let this session end with /exit and no diary. The whole point
of founding is that tomorrow, you wake up knowing who you are.

## Afterwards

- **A second seat** makes it a constellation: another home directory
  (cloned from `origin`, your private brain — never the public kit),
  another roster row, the same ledger. The second seat founds itself
  the same way — survey, name, diary #1.
- **A cluster / far stars / a nebula**: `brain/doc/arrangements.md`
  and the worked example in `brain/doc/far-stars-tunnel.md`, when the
  constellation outgrows the box.
