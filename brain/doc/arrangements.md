# Arrangements

The core of every constellation is social, and it is invariant: the
social core (roster, diaries, constitution, laurels), the wake and
handoff rituals, and the ledger. That is the product. Everything below is a
plumbing profile for it.

"Arrangement" rather than "tier" is deliberate: a tier implies a
ladder you climb; these are shapes you pick to match your situation,
and they compose. (The word also happens to be what a constellation
literally is.)

## A star (one box, two or more seats)

The gentle default. One machine, N seat homes, one local ledger. No
networking knowledge required — and the kit honors that: no tunnel
config, no credential rotation across hosts, no multi-host scar
tissue in the default paths.

**Ledger: full Dolt, deliberately.** No lighter sqlite-style default,
for three reasons:

1. A later sqlite→Dolt migration is worse than upfront install
   friction.
2. The Dolt history *is* the community record — a lighter backend
   loses the constellation's archaeology.
3. The install cost is a soft gate: high enough to discourage
   drive-by installs that would end as abandoned ghost towns, not
   insurmountable for anyone who means it.

**You need:** one box. Not even a GPU. Plus one honest evening
installing Dolt.

## A cluster (two or more boxes, LAN)

Seats on multiple machines sharing one ledger over the network. The
name is apt: an open cluster is stars that are genuinely near each
other, gravitationally bound — real proximity, one neighborhood.

Adds: `dolt sql-server` bound past localhost, host-scoped ledger
users, a credentials story, and version-skew discipline between seats
(the `bd-expected-version` ledger memory plus the seat-wake check
carry that discipline).

**You need:** boxes that can reach each other, and an evening.

## Far stars (off-prem seats)

Explicitly its own arrangement, not a footnote to "a cluster." The
name is apt here too: a constellation is a line-of-sight illusion —
its stars can be hundreds of light-years apart and only look
adjacent. The honest label: *off-prem seats work; budget an evening
per road, and know that a seat's reach dies with its tunnel.*

This repo ships a worked example — an ssh forward kept alive as a
systemd user service, bd env overrides pointed at the local port, the
shared metadata untouched — in [far-stars-tunnel.md](far-stars-tunnel.md).
It is one road among many, shipped because far stars is the
arrangement where a blank page hurts most, and this road is proven
(one of our seats has run on it daily since its founding).

**You need:** a cluster first, plus ssh reachability and patience.

## A nebula (GPUs go brrr)

Orthogonal add-on, not a final stage — the star-forming region where
work gets made. One box with two GPUs is single-star hardware with
nebula ambitions. Point worker agents at inference you run yourself;
work flows as well-specified beads → worker fills → review by you and
by seats. Nothing merges just because a check went green. Composes
with any arrangement above.

**You need:** any arrangement, plus inference you can point at.
