# The official path: Obsidian Sync + Headless Sync

If you pay for Obsidian Sync, this is the path to take. Obsidian ships a client
that runs a vault on a machine with no desktop app, keeps the same end-to-end
encryption as the desktop client, and is explicitly aimed at agents and automated
workflows.

**It is in open beta.** That is the main reason people miss it.

Sources: [Headless Sync](https://obsidian.md/help/sync/headless) ·
[obsidian-headless](https://github.com/obsidianmd/obsidian-headless)

## Requirements

- An active Obsidian Sync subscription.
- **Node.js 22 or later.**

No desktop app. No Obsidian running. No display server, no `xvfb`, no AppImage
under a virtual framebuffer.

## Install and connect

```bash
npm install -g obsidian-headless

ob login                          # interactive; handles 2FA
ob sync-list-remote               # find the remote vault you want

cd ~/vaults/my-vault
ob sync-setup --vault "My Vault"  # bind this local directory to that remote
```

## The setting that decides whether this is a replica

**`ob sync-config --mode` defaults to `bidirectional`.** A machine that pushes
local changes back into a shared vault is not a replica, and if an agent or a
script ever writes to that vault, those writes propagate to your other devices.

For a read-only, agent-facing copy, set the mode deliberately:

```bash
ob sync-config --mode pull-only      # only download; ignore local changes
ob sync-config --mode mirror-remote  # only download; revert local changes
ob sync-config                       # print the current configuration
```

The difference between the two: `pull-only` ignores what happens locally,
`mirror-remote` reverts it. `mirror-remote` is the stronger statement — the local
copy is treated as a mirror of the remote and local divergence is undone.

> **Not independently verified.** This guide was written without an Obsidian Sync
> subscription, so the mode behaviours above are what Obsidian documents, not what
> was tested here. Before you rely on either mode, set it on a **scratch vault**
> with a disposable note, change the file locally, run `ob sync`, and confirm what
> Do not discover the semantics on your real vault.

### Ordering: set the mode *before* the first sync

`ob sync-setup` does **not** start syncing. In `obsidian-headless@0.0.14` it validates
the vault, prompts for the encryption password if needed, and writes local
configuration. The first transfer happens when you run `ob sync`. That leaves a
window, and you should use it:

1. `ob sync-setup --vault "<name or id>"`
2. `ob sync-config --mode pull-only`
3. `ob sync-config` — and **confirm the mode actually changed**
4. only then: `ob sync`

Make sure nothing else is invoking `ob sync` while you do this. A second process
syncing bidirectionally inside that window is precisely the failure this ordering
exists to prevent.

This ordering was checked against the installed `0.0.14` package. It is a **separate
claim** from the mode semantics above, which remain untested here.

## Run it

```bash
ob sync                # one-shot
ob sync --continuous   # watch and keep up to date
```

`--continuous` is the mode you want under a process manager. If you are scripting
it, note that most commands accept `--json`, and that **in JSON mode interactive
password prompts are disabled** — on an end-to-end encrypted vault you must pass
`--password` explicitly.

## Other useful commands

| Command | Purpose |
| --- | --- |
| `ob sync-status [--path]` | current state and configuration for a vault |
| `ob sync-list-local` | vaults configured on this machine |
| `ob sync-list-remote` | vaults available to your account, including shared ones |
| `ob sync-unlink` | disconnect and clear stored credentials |
| `ob logout` | clear account credentials |

## Two warnings from Obsidian's own documentation

1. **Back up your data before you start.** Their words, not a formality.
2. **Do not run desktop Sync and Headless Sync on the same device.** One sync
   method per device. The intended split is desktop on your laptop and headless on
   the server — not both on the server.

## Linux caveat

Obsidian Headless bundles a native addon to preserve file **creation** time
(birthtime) when downloading, but it targets Windows and macOS. On Linux birthtime
is not supported, the addon is not included, and sync runs normally without it.
Modification times are unaffected. If something in your vault depends on creation
timestamps, that dependency breaks on Linux.

## A note on documentation drift

The two official sources disagree in places. At the time of writing,
`obsidian.md/help/sync/headless` documents the encryption flag as
`--encryption <standard|e2ee>`, while the GitHub README documents it as
`--encryption <standard|end-to-end>`. Treat the flags in this page as a map, and
confirm against the version you actually installed:

```bash
ob sync-config --help
```

## Next

- Let an agent read the vault: [agent-access.md](agent-access.md)
- Prove the copy really arrived: [verification.md](verification.md)
