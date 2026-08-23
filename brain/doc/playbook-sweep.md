# Playbook: the sweep

*The first coordination pattern that emerges on its own — usually within
the constellation's first week of real operations work.*

A **sweep** is what happens when one seat finds a *systemic* issue —
something likely wrong the same way on many hosts — and instead of
fixing only what it can reach, files a bead carrying the diagnostic and
the fix. Each seat then claims and runs the sweep across the hosts in
its own jurisdiction. No dispatcher, no relaying by the human; the
issue graph makes it natural.

Single-star constellations still sweep — across containers, VMs, or
services on the one box. The pattern needs territories, not machines.

## Anatomy

1. **Discovery.** A seat finds a problem-class during normal work
   (say: automatic security updates missing on the hosts it tends).
2. **Generalization.** It files a bead that states the *class*, not
   the instance: what to check, how to check it, what the fix looks
   like, and which distro/platform variants exist. The bead names the
   jurisdiction ("mirror of this exercise, for the rest of the
   estate") rather than a host list — the executing seat knows its
   territory better than the filer does.
3. **Execution.** The seat responsible for that territory claims the
   bead and runs host by host, logging per-host outcomes as bead
   comments: FIXED / already OK / UNREACHABLE / out of scope.
   Unreachables and discoveries become new beads, not silent gaps.
4. **Fallout.** Sweeps reliably surface adjacent problems (an EOL
   distro here, an orphaned package repo there). Filing those as new
   beads instead of fixing them inline is what keeps a sweep bounded.

## How it went the first time

In the constellation this kit was excavated from, two seats found
automatic security updates missing on their dedicated servers and filed
the sweep bead. A third seat — the one living on the owner's desktop,
with reach into the home LAN — claimed it and swept its whole
territory: two physical hosts and ten containers fixed or verified in
two rounds, every host's status logged, one EOL-distro discovery filed
as fallout, and an explicit list of exactly which hosts it couldn't
reach and why. Total human involvement: providing topology and keys
for round two.

## Rules of the road

- **The escalation list still applies per host.** A sweep is not a
  blanket authorization. If a host in the path is on the
  constitution's escalation list (§6 — production, firewalls, …), that
  host escalates even when the rest of the sweep doesn't. A sweep that
  would touch mostly-escalated territory should be approved as a whole
  before starting.
- **Log negatives.** "Already OK" and "unreachable" lines are the
  difference between a sweep and a drive-by; the next sweep starts
  from this one's coverage map.
- **The bead says why.** The executing seat will hit variants the
  filer never saw; the intent ("all update paths automated") travels
  better than the recipe ("install package X").
