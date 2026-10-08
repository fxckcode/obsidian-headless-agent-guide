# Verifying it actually works

A green sync icon is not evidence. Neither is a synced folder that "looks right".
This page is how you turn a plausible setup into a known one.

## 1. Count and measure both sides

On the source and on the server, compare the same numbers:

```bash
find "$VAULT" -name '*.md' | wc -l     # note count
du -sh "$VAULT"                        # total size
```

They should match. If they differ, stop and find out why before you wire an agent
to the replica. "Close enough" is how people spend an evening reading half a vault
and not noticing.

## 2. Check one file byte for byte

Counts can match while content is stale. Pick a note you edited recently and
compare hashes across the two machines:

```bash
sha256sum "$VAULT/PROJECTS/example.md"
```

A mismatch with matching counts means you are looking at an older revision of a
correct-looking vault.

## 3. Check for conflict artifacts

Conflict files are the quiet failure mode. They do not break anything, they just
accumulate, and they mean your two writers disagreed.

```bash
find "$VAULT" -name '*conflict*' -o -name '*sync-conflict*'
```

An empty result is what you want. If you get hits, read them before deleting:
they contain the losing version.

## 4. Verify through the protocol, not the filesystem

The filesystem being correct does not prove the MCP server works. Drive it:

```bash
printf '%s\n' \
  '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18"}}' \
  '{"jsonrpc":"2.0","id":2,"method":"tools/list"}' \
  | <your-mcp-command> <vault-path>
```

A handshake response and a tool list means the server starts, talks MCP, and can
find the vault. Then pick one tool from that list and call it with `tools/call` to
confirm it reads real content.

If this returns nothing, the problem is your MCP configuration or the vault path —
not the transport. If the tool list appears but tool calls return empty, the
server is running against an empty or wrong directory.

## 5. Confirm the writer count

You designed for exactly one writer: the transport. Verify it:

- The MCP server is read-only, in the version you installed.
- The server-side syncer is one-way.
- Nothing else has the vault directory mounted read-write.

## The traps, in detail

**Read-only does not protect you from the syncer.** Your MCP configuration
constrains the agent. It does not constrain Syncthing, Headless Sync, or a stray
`git` command. Those are separate writers with their own rules.

**Receive-only reports out-of-sync on the *other* devices.** If you look at the
replica and it says "up to date", that is expected and says nothing about whether
the cluster has newer data. Look at the sending device.

**A replica is not a backup.** Deletions propagate. If a note is deleted on the
source, the replica deletes it too. Keep a real backup somewhere the transport
cannot reach.

**`.obsidian/` is device state.** It carries plugin configuration, workspace
layout, and sometimes credentials. Decide explicitly whether it crosses the
network, and keep it out of what you hand an agent. Obsidian writes to it
constantly, so including it also generates pointless sync churn.

**Timestamps differ by platform.** Headless Sync on Linux does not preserve file
creation time. Anything that depends on birthtime is broken on Linux, silently.

## Recovery

Know which button does what *before* you need it, because one of them is
destructive:

**Syncthing, receive-only replica — "Revert Local Changes."** Targeted. It deletes
files that were added locally and re-syncs modified or deleted ones from the
cluster. This is the button you want when something wrote to the replica.

**Syncthing, send-only side — "Override Changes."** Destructive to the rest of the
cluster. It enforces *this host's* state everywhere, overwriting files and deleting
anything that does not exist locally. Do not press it to "fix" a conflict until
you are certain this host is the correct version.

**Headless Sync.** `ob sync-status` shows state and configuration. `ob sync-unlink`
disconnects the vault and clears stored credentials, which is the clean way to
detach a machine.

## A checklist worth keeping

- [ ] Source and replica agree on note count and size.
- [ ] A spot-checked note hashes identically on both sides.
- [ ] No conflict artifacts present.
- [ ] MCP server handshakes and lists tools over stdio.
- [ ] A tool call returns real content.
- [ ] The MCP server runs read-only, verified in the installed version.
- [ ] The syncer on the server is one-way.
- [ ] A real backup exists that the transport cannot reach.
- [ ] `.obsidian/` handling is a deliberate decision, not an accident.
