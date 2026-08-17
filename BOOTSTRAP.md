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

Inventory before changing anything: OS, whether `bd` is installed and
its version, and whether this repo clone is where the human wants
the constellation's brain to live. Record the survey in `brain/doc/` — it is
the first page of institutional memory.

If beads is missing, install it **from its current upstream
instructions**, not from memory:

- beads: https://github.com/gastownhall/beads

(You do not need a separate Dolt install for founding — `bd` embeds its
own Dolt storage. Install the Dolt CLI only if a later arrangement
calls for it.)

Two known installer traps (observed with the 1.2.2 installer, verify
against current behavior):

- **`scripts/install.sh` can exit nonzero after a SUCCESSFUL install**
  when `~/.local/bin` is not yet on PATH — it prints the PATH advice,
  then errors anyway. Any `set -e` founding script dies right there
  with a working `bd` on disk. Check for the binary before concluding
  the install failed.
- **Fork detection can silently reroute every write.** If this clone's
  remotes look fork-shaped (e.g. `origin` = your private remote,
  `upstream` = the public kit — exactly what Step 2 creates), `bd init`
  may decide you are a contributor, write `beads.role=contributor` to
  **global** git config without saying so, and route all writes to a
  side repo under `~/.beads-planning`. If that dir is later deleted,
  writes fail with the misleading `embeddeddolt: store is read-only`,
  and the setting survives `rm -rf .beads` because it lives in git
  config. Found a founding: run
  `git config beads.role maintainer` before `bd init`, or pass
  `bd init --role maintainer`, and verify afterwards with
  `git config --get-all beads.role`.

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

The ledger is `bd`'s **embedded** Dolt database, living in this repo's
`.beads/` and synced through your git remote. There is no server to
run for a star — issues live in the local DB, sync uses
`refs/dolt/data` on `origin`, and `.beads/issues.jsonl` is a passive
export, not the database. (An external `dolt sql-server` is a
*multi-seat arrangement* — several seats sharing one ledger over
localhost or tunnels — added later per `brain/doc/arrangements.md`,
not part of founding. Earlier kit revisions founded on server mode;
that path is stale: `dolt sql-client` was removed in dolt 1.88, and
`bd init --server` writes have failed with
`embeddeddolt: store is read-only` on current bd. Do not fight it —
found embedded.)

1. In this repo (with `beads.role maintainer` set — see the Step 1
   trap): `bd init`, accepting the embedded default. No credentials,
   no port, no service. If any invocation ever asks for a Dolt
   password (server arrangements later), pass it as the
   `BEADS_DOLT_PASSWORD` env var or a wrapper env file — the
   `~/.config/beads/credentials` file is not honored by current bd
   (1.2.2), and a founding-era constellation lost an evening to a
   hand-authored credential typo.
2. Merge `templates/beads-config.yaml` into the generated
   `.beads/config.yaml` and commit it — it disables per-write
   auto-backup (backups are a host concern, not a write-path side
   effect; document your dump/sync schedule in
   `brain/doc/ledger-ops.md`).

## Step 4 — Wire sync and prove the round trip

- Wire the ledger to your private remote per current beads docs
  (`bd dolt push` / `bd dolt pull` against `origin`'s
  `refs/dolt/data`). Push once and confirm the ref exists:
  `git ls-remote origin 'refs/dolt/*'`.
- **Sync caveat for memories:** some bd versions never stage or commit
  the Dolt `config` table that `bd remember` writes to, so a
  commit-level sync silently omits every memory saved since the last
  manual commit. Check with
  `bd sql "SELECT table_name, staged, status FROM dolt_status WHERE table_name = 'config'"`
  — if it shows `modified`, run, as two separate commands,
  `bd sql "CALL dolt_add('config')"` then
  `bd sql "CALL dolt_commit('-m','commit config table')"`
  before (or on a schedule alongside) any `bd dolt push`.
- Record the constellation's bd version in ledger memory under the well-known
  key: `bd remember --key bd-expected-version "..."` — and write the
  same version into `.claude/skills/seat-wake/bd-baseline`. Seat-wake
  compares against both at every wake; upgrading bd is thereafter a
  deliberate constellation-wide act (bump the memory, roll the seats).
- **Round-trip test:** create a scratch bead, read it back, update it,
  close it. From a *second shell*, confirm the same. Then `bd dolt
  push`, and from a scratch clone `bd dolt pull` and read the bead
  back — sync you haven't proven is sync you don't have.
- If a write ever fails with `embeddeddolt: store is read-only`, the
  message is misleading: check `git config --get-all beads.role`
  first (Step 1's contributor-routing trap), not filesystem
  permissions.

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
