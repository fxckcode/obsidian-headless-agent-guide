# Choosing a transport

Four options that actually work, what each one costs, and how to pick.

## The comparison

| | Official Sync + Headless | Syncthing | LiveSync + CouchDB | git |
| --- | --- | --- | --- | --- |
| **Cost** | Obsidian Sync subscription | free | free + a backend to run | free |
| **End-to-end encryption** | yes (Sync's E2EE) | not by default (TLS transport only) | supported, plugin-provided | no (unless you self-host the remote over SSH) |
| **Server keeps a copy** | yes | yes | yes, in a database or object bucket | yes, in the repo |
| **Mobile clients** | yes | awkward | yes, this is its strength | awkward |
| **Real-time** | near-real-time | near-real-time | yes | no |
| **Conflict handling** | Sync's own | file-level; conflict copies | plugin-level smart merge | manual |
| **What runs on the server** | Node 22+ | one daemon | CouchDB, or object storage (S3/MinIO/R2); peer-to-peer needs no central server | `git pull` on a timer |
| **Maturity** | official, **open beta** | mature | mature, community (12k+ stars, since 2021) | mature |
| **Main thing that will annoy you** | it is beta, and the mode defaults to bidirectional | the sync status is not where you think it is | a second system to keep alive and backed up | conflicts are your problem, and it syncs on commit, not on save |

## How to pick

```text
Do you already pay for Obsidian Sync?
├── yes ──► Official Sync + Headless Sync
│           docs/official-sync.md
│           (and set the mode off its bidirectional default)
└── no
    ├── Do you need your notes on a phone?
    │   ├── yes ──► Self-hosted LiveSync + CouchDB
    │   └── no
    │       ├── Is your vault mostly plain text, without huge attachments?
    │       │   ├── yes ──► Syncthing, one-way
    │       │   │           docs/syncthing.md
    │       │   └── no ───► Syncthing still, but watch the transfer volume
    │       └── Do you want a full version history and are comfortable resolving
    │           merge conflicts by hand? ──► git
```

## Notes on each, in the order you should consider them

**1. Official Sync + Headless Sync.** If you already pay, this is not a
competition: it is the vendor-supported path, it carries your existing E2EE, and
it was explicitly built for agents and automated workflows. Its two drawbacks are
that it is in open beta and that its default sync mode is `bidirectional`, which
you must change for a replica. See [official-sync.md](official-sync.md).

**2. Syncthing, one-way.** The best self-hosted answer when you have no mobile
requirement: one daemon, no database, no plugin, plain files, and a `receive only`
folder type that makes the server structurally unable to push changes back. See
[syncthing.md](syncthing.md).

**3. Self-hosted LiveSync + CouchDB.** Choose this when mobile matters. The
backends are CouchDB *or* object storage (S3, MinIO, R2), and it can also run
peer-to-peer over WebRTC with no central data server — though peer discovery still
needs a signalling relay, and at least one device holding the data must be online.
You are trading a daemon for a backend: it becomes something else to run, patch,
secure and back up, and every device gets a plugin. The payoff is real-time sync
with plugin-level merge conflict resolution. It is the most mature community
option by a wide margin, and it is **not compatible with Obsidian Sync** — do not
enable both.

**4. git.** A perfectly good answer that people forget. The `obsidian-git` plugin
commits and pushes; the server pulls on a timer or a webhook. It is not real-time,
it does not handle binary attachments elegantly, and merge conflicts are manual —
but you get complete history and a transport you already understand. Not
independently tested for this guide.

## What all four have in common

- **They are transports. None of them is an agent.** Getting the files to the
  server and letting a program read them are separate layers; conflating them is
  the most common structural mistake in these setups.
- **They all write to your vault directory.** That is what the MCP layer has to be
  designed around. See [agent-access.md](agent-access.md).
- **None of them replaces a backup.** A replica is not a backup. If a file is
  deleted on the source and the deletion syncs, it is gone from the replica too.

## Next

- [agent-access.md](agent-access.md) — letting an agent read the vault
- [verification.md](verification.md) — proving it actually works
