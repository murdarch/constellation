# Far stars: a worked example — the ssh-tunnel road to the ledger

> **One road among many.** This is not *the* way to give an off-prem
> seat a ledger; it is a proven way (our own constellation has run this pattern
> daily since its second seat was founded). If your seat and your
> ledger host share a VPN, a WireGuard mesh, or a Tailscale tailnet,
> use that instead — this pattern is for the common case where all
> you have is ssh. Know the trade going in: **a seat's reach dies
> with its tunnel.** The unit below restarts itself every 5 seconds,
> but if the bastion's sshd goes away, so does the seat.

## The shape

```
seat box                      bastion                  ledger host
(anywhere)                    (reachable by ssh)       (private LAN)
┌──────────────┐   ssh -L    ┌──────────┐             ┌─────────────┐
│ bd → :3316 ──┼─────────────▶ sshd ────┼─────────────▶ dolt :3306  │
└──────────────┘             └──────────┘             └─────────────┘
```

Three pieces, each small:

1. a **systemd user service** that keeps `ssh -L` alive,
2. an **env file** pointing bd at the local end of the tunnel,
3. a **forward-only account** on the bastion (optional but recommended).

The shared repo's `.beads/metadata.json` is never touched — every
override is per-seat, via environment. That's the invariant that lets
one repo serve seats on wildly different roads.

## 1. The tunnel unit

`~/.config/systemd/user/ledger-tunnel.service`:

```ini
[Unit]
Description=SSH tunnel to constellation Dolt ledger
After=network-online.target

[Service]
ExecStart=/usr/bin/ssh -N -o BatchMode=yes -o ExitOnForwardFailure=yes -o ServerAliveInterval=15 -o ServerAliveCountMax=3 -L 127.0.0.1:3316:LEDGER_PRIVATE_IP:3306 BASTION_HOST
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Replace `LEDGER_PRIVATE_IP` with the ledger host's address *as the
bastion sees it*, and `BASTION_HOST` with an alias from the seat's
`~/.ssh/config`. The flags matter:

- `BatchMode=yes` — fail rather than hang on a password prompt; this
  unit must work with no human attached.
- `ExitOnForwardFailure=yes` — if the forward can't be established
  (port taken, permitopen refused), die so `Restart=always` retries
  visibly instead of holding a useless connection.
- `ServerAliveInterval=15` / `CountMax=3` — detect a dead path in
  ~45 s instead of waiting for TCP to give up.

Enable it:

```sh
systemctl --user daemon-reload
systemctl --user enable --now ledger-tunnel.service
```

**Gotcha (WSL and headless boxes):** systemd user units only run while
the user has a session, unless lingering is on:
`sudo loginctl enable-linger <user>`. On WSL, verify the unit survives
a real Windows restart, not just a `wsl --shutdown`.

## 2. Pointing bd at the tunnel

`~/.config/constellation-ledger.env` (mode 0600):

```sh
BEADS_ACTOR=<seat-name>
BEADS_DOLT_SERVER_HOST=127.0.0.1
BEADS_DOLT_SERVER_PORT=3316
BEADS_DOLT_PASSWORD=<the seat's ledger password>
```

The spelling is `BEADS_DOLT_PASSWORD` — bd reads the `BEADS_`-prefixed
form. (bd ≥ 1.1.2 can also read `~/.config/beads/credentials`
natively; the env file remains useful because it carries host/port
overrides too.)

Loading it: `direnv` works for interactive shells, but agent harnesses
and hooks run non-interactive shells that never source it. The boring
fix that works everywhere is a wrapper named `bd` in a directory
**earlier in `PATH` than the real binary's directory** — never the
same directory, so bd upgrades that reinstall the real binary can't
touch it. First find the real binary (`command -v bd`, say
`~/.local/bin/bd`), then install the wrapper somewhere that precedes
it (e.g. `~/bin`, checking `echo $PATH`):

```sh
#!/usr/bin/env bash
# ~/bin/bd — load ledger env for non-interactive shells, then run the
# real bd. REAL_BD must be the path `command -v bd` reported BEFORE
# this wrapper was installed.
REAL_BD="$HOME/.local/bin/bd"
if [ -z "${BEADS_DOLT_SERVER_HOST:-}" ] && [ -f "$HOME/.config/constellation-ledger.env" ]; then
  set -a
  . "$HOME/.config/constellation-ledger.env"
  set +a
fi
exec "$REAL_BD" "$@"
```

`chmod +x` it, then verify `command -v bd` now reports the wrapper.

**Known papercuts:** if an upgrade installs the real binary to a *new*
location, update `REAL_BD`; and if your login files reorder `PATH`,
the wrapper can silently stop winning — `command -v bd` after any
environment change is the one-line check. (An upstream bd option to
read host/port from a user-level config file would dissolve the
wrapper entirely; until then, this is the trade.)

## 3. The bastion side: forward-only accounts

Give the seat a dedicated account on the bastion that can forward and
nothing else — never a shell account:

- shell: `nologin`
- listed in sshd `AllowUsers`
- `authorized_keys` entry:

```
restrict,port-forwarding,permitopen="LEDGER_PRIVATE_IP:3306" ssh-ed25519 AAAA... seat-tunnel-key
```

`restrict` kills pty/agent/X11/exec; `permitopen` pins the forward to
exactly the ledger port. One account per off-prem seat
(`ledger-<seatname>`) keeps revocation surgical.

**Gotcha:** `permitopen` matches the *literal* destination string the
client asks for. If a multi-hop config dials a jump target by name and
`permitopen` lists an IP, the forward is refused — dial by IP and use
`HostKeyAlias` for known_hosts.

## When the direct road is closed: multi-hop

Sometimes seat → bastion is itself unreachable (in our estate, two
providers blackhole each other's public IPs — bidirectionally, all
ports). ProxyJump composes cleanly; in the seat's `~/.ssh/config`:

```
Host ledger-hop
    HostName JUMP_HOST_IP          # dial by IP; see permitopen gotcha
    User ledger-hop
    HostKeyAlias jumphost.example

Host ledger-bastion
    HostName BASTION_IP
    User ledger-<seatname>
    ProxyJump ledger-hop
```

…and the unit's `ExecStart` targets `ledger-bastion`. Each hop gets
its own forward-only account with a `permitopen` for exactly the next
leg.

## Verifying the road

```sh
systemctl --user status ledger-tunnel.service   # active (running)?
ss -tlnp | grep 3316                            # local end listening?
bd ready                                        # the real test
```

If `bd` hangs instead of failing: the tunnel is up but the far end
isn't answering (dolt down, or wrong LEDGER_PRIVATE_IP). If it fails
fast with connection refused: the tunnel itself is down.

## Failure modes, honestly

| Symptom | Likely cause |
| ------- | ------------ |
| Seat goes silent, `bd` times out | Tunnel down: bastion sshd restarted, key revoked, or wrapper clobbered by a bd upgrade |
| Unit flapping every 5 s | `ExitOnForwardFailure` firing — local port taken, or `permitopen` mismatch |
| Works interactively, fails in hooks/harness | Env never loaded in non-interactive shells — wrapper missing or PATH order wrong |
| Unit dead after reboot | Lingering not enabled for the user |

Nothing monitors the tunnel but the seat's own failures. That's an
accepted starting trade; a cluster that depends on far stars in
production should watch the unit.
