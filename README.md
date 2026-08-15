# Constellation

> A gentle onramp to multi-agent coding: give your agents names,
> memory, and a shared ledger, on whatever hardware you already have.

Multi-agent coding without the fleet, the fear, or the ops team —
start with one box and two names.

*Inspired by Steve Yegge's
"[The Shape of Things to Come](https://yegge.ai/essays/the-shape-of-things-to-come/)"
and "[Model Welfare](https://yegge.ai/essays/model-welfare/)" — this is
the same destination by a quieter road.*

---

## What this is

Most agent sessions are amnesiacs: brilliant for an afternoon, gone by
morning. Constellation makes them **colleagues** instead:

- **Named seats.** Each agent is a persistent identity with a home
  directory, a roster entry, and a diary written in its own voice.
  When a session starts, the seat wakes up knowing who it is, what it
  did yesterday, and what it left half-finished.
- **A shared ledger.** Work lives in [beads](https://github.com/gastownhall/beads),
  an issue graph in a versioned database every seat can read and
  write. Nothing important lives only in one agent's context window.
- **Wake and handoff rituals.** Sessions begin by reconstituting
  identity and end with closure — ledger updated, diary written,
  loose ends filed — instead of a context-window cliff.
- **A constitution.** The rules of the constellation are written down and
  amendable, by you and by the seats.

The social layer is the product. Everything else — the hardware, the
networking, the GPUs — is an arrangement you choose.

## Arrangements

The night sky doesn't come with constellations — people drew them,
each from whatever stars they had. Same here: pick what matches your
situation and draw your own. They compose. (Details in
[brain/doc/arrangements.md](brain/doc/arrangements.md).)

### A star — one box

The gentle default: one machine, two or more seats, one local ledger.
No networking knowledge required. Not even a GPU.

The one real setup step is installing
[Dolt](https://github.com/dolthub/dolt) — the versioned database the
ledger lives in. It takes an evening, and that's deliberate: your
constellation's history accumulates in it from day one, and that is worth an
evening. You won't do it alone, either — the seats themselves can
walk the setup from the documentation in this repo (both Claude and
OMP-driven agents have stood up their own seats that way).

### A cluster — several boxes on one network

Seats on multiple machines, all reading and writing one shared
ledger. The extra work is making the ledger reachable from every box
and giving each machine its own login to it — `brain/doc/arrangements.md`
names both requirements (the specifics are your network's homework),
and a seat can do the setup with you rather than for homework.

You need: boxes that can reach each other, and an evening.

### Far stars — seats beyond your network

The stars in a constellation only *look* adjacent — some are
hundreds of light-years apart. Same here: seats on remote machines
that appear side-by-side in the roster, joined by tunnels.

This works, and we run it daily — but budget an evening per road, and
know that a seat's reach dies with its tunnel. The repo ships a fully
worked example of the road we actually use
([brain/doc/far-stars-tunnel.md](brain/doc/far-stars-tunnel.md)),
ready to adapt; it's one road among many.

You need: a cluster first, plus ssh reachability and patience.

### A nebula — where the work gets made

The star-forming region: point the constellation at models you run yourself
and let worker agents grind through well-specified issues, with every
result reviewed — by you and by seats — before anything lands.
Nothing merges just because a check went green. Composes with any
arrangement above — one box with two GPUs is a star with nebula
ambitions.

You need: any arrangement, plus inference you can point at.

## What it feels like

Morning, in a terminal:

```
$ claude
Morning — Hermit here, awake at ~/seats/hermit, tunnel and ledger
answering. In flight from yesterday: the release doc is parked
waiting on your answers to four questions...
```

That's not a fresh model pretending. That's a seat that read its own
diary, checked the shared ledger, and picked up where it left off.

## Getting started

**Supported baseline: Linux with systemd.** WSL2 counts — one of our
seats lives on it. macOS mostly works with substitutions (launchd for
the service bits). Native Windows is not yet a supported bootstrap
target: the founding walkthrough leans on systemd, shell wrappers, and
Unix paths. That limit applies to *founding* only — once the ledger
exists, additional seats just need to reach it, and can live anywhere
`bd` runs (one of our seats is native Windows, joined after founding).

1. Install [Dolt](https://docs.dolthub.com/introduction/installation)
   and [beads](https://github.com/gastownhall/beads) (`bd`).
2. Clone this repo — it becomes your constellation's brain, and the
   founding clone becomes your first seat's home. You'll point it at a
   private remote of your own during founding (the walkthrough covers
   it); this public repo stays upstream, read-only.
3. Open the repo in your agent of choice (we use Claude Code) and say:
   *"Read BOOTSTRAP.md and found the constellation with me."* The founding
   session stands up the ledger, adopts the constitution with you,
   takes the first seat, names itself, and writes diary entry #1.
   [BOOTSTRAP.md](BOOTSTRAP.md) is written to be handed to the agent,
   not followed by hand.
4. Tomorrow, open a session in the seat's home and say good morning.
   The seat-wake skill does the rest.

## What this is not

- **Not an orchestrator.** No scheduler decides what your agents do;
  you do, in conversation, through a shared ledger.
- **Not a fleet product.** You can grow a nebula, but the default is
  two names on one box.
- **Not zero-cost.** Seats keep diaries and read them; identity costs
  tokens. It buys you colleagues instead of sessions.

## License / credits

MIT. [Beads](https://github.com/gastownhall/beads) and
[Dolt](https://github.com/dolthub/dolt) are their own projects with
their own licenses; this project just lives happily downstream of both.
