# Obsidian on a headless server, readable by an agent

Your vault lives on a machine with no desktop. You want an agent — Claude, Codex,
Hermes, anything that speaks MCP — to be able to read it.

That is **two problems**, and most broken setups are broken because they were
treated as one:

1. **Transport.** Getting the files onto the server in the first place.
2. **Access.** Letting an agent read them safely once they are there.

This guide covers both, names the traps, and tells you which existing tool to use
instead of pointing you at a new one to maintain.

## Three things this guide will not do

- It will not pretend your notes must go through an MCP server to be synced. They
  are separate layers with separate failure modes.
- It will not recommend running two sync systems over one vault.
- It will not claim that a read-only agent surface protects your files from the
  sync process. It does not. See [verification](docs/verification.md).

## Start with the decision tree

**Do you pay for Obsidian Sync?**

- **Yes** → [docs/official-sync.md](docs/official-sync.md). Obsidian ships a
  headless client that syncs a vault with no desktop app. This is the path most
  people are missing, because it is still in open beta.
- **No** → pick from [docs/transport-options.md](docs/transport-options.md):
  Syncthing (simplest, no server component), Self-hosted LiveSync + CouchDB
  (mobile and real-time), or plain git.

**Then, whatever you chose** → [docs/agent-access.md](docs/agent-access.md) for
the MCP layer, and [docs/verification.md](docs/verification.md) before you trust
any of it.

The full comparison, including what each option costs you, is in
[docs/transport-options.md](docs/transport-options.md).

## The traps, collected

Each of these has burned someone. They are expanded where they belong, but you
should know they exist before you start:

- **Headless Sync defaults to `bidirectional`.** A "replica" that pushes changes
  back is not a replica. Set the mode deliberately.
- **Use one sync method per device.** Obsidian's own documentation is explicit
  that running desktop Sync and Headless Sync on the same device causes conflicts.
- **Do not layer two sync services over one vault.** Obsidian's own FAQ does not
  recommend running a third-party sync service alongside Obsidian Sync: across
  multiple devices it leads to conflicts and to duplicate or corrupted files.
  Self-hosted LiveSync is blunter — it states it is not compatible with Obsidian
  Sync at all. Pick one method per vault.
- **On-demand / "online-only" cloud files can delete your notes.** This is the
  nastiest trap here. If a cloud client (OneDrive, Dropbox) is holding files as
  online-only, Obsidian Sync sees them as missing and **removes them from your
  remote vault**. Turn off on-demand downloads before you combine any of this.
- **Obsidian Sync on Linux has no birthtime.** File creation times are not
  preserved on Linux; modification times are.
- **A read-only agent cannot corrupt your vault, but your syncer still can.** The
  agent surface and the sync daemon are independent writers. Only one of them is
  constrained by your MCP configuration.
- **Syncthing receive-only does not report "out of sync" locally.** It is the
  *other* devices that see the replica as behind. Looking at the wrong dashboard
  tells you the wrong thing.

## What already exists

This guide is deliberately written so that most of its advice ends in "use the
thing that exists". Before you build anything, know what is out there:

| Need | What exists |
| --- | --- |
| MCP server for a local vault | [`@bitbonsai/mcpvault`](https://github.com/bitbonsai/mcpvault) — the most established option, npm-installable, documents a read-only mode |
| MCP server, no Node runtime | [`lstpsche/obsidian-mcp`](https://github.com/lstpsche/obsidian-mcp) (Rust), [`nsega/mcp-obsidian`](https://github.com/nsega/mcp-obsidian) (Go) — both ship a single binary |
| MCP server inside Obsidian | the [Local REST API](https://github.com/coddingtonbear/obsidian-local-rest-api) community plugin now ships one, but it needs the desktop app running |
| Vault on a headless server, paid tier | [`obsidian-headless`](https://github.com/obsidianmd/obsidian-headless) — Obsidian's own client |
| Vault on a headless server, self-hosted | [Syncthing](https://syncthing.net/), [Self-hosted LiveSync](https://github.com/vrtmrz/obsidian-livesync) |

The single-binary options above are small projects. That is not a criticism, it is
a fact you should weigh when you decide what to run unattended against your notes.
The details are in [docs/agent-access.md](docs/agent-access.md).

## Scope

This is a guide, not a product. It carries no daemon, no plugin, and no server to
keep patched. Where it describes a procedure that was run and verified on a real
host, it says so; where it describes something only documented upstream and not
independently tested, **it says that too**.

## Contributing

Corrections are the most valuable contribution — see [CONTRIBUTING.md](CONTRIBUTING.md).
Participation is covered by the [Contributor Covenant](CODE_OF_CONDUCT.md).

## License

MIT. See [LICENSE](LICENSE).
