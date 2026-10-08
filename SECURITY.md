# Security policy

This repository is documentation. It ships no executable code, so the
vulnerabilities that matter here are not the usual kind.

## What counts as a security issue here

- **Instructions that could destroy data.** A procedure that deletes, overwrites,
  or silently reverts notes when followed as written. The destructive button
  warnings in the transport docs exist for this reason.
- **A recommendation that exposes a vault.** Advice that results in a vault, its
  configuration, or its credentials being reachable by someone who should not have
  them — a published admin port, a network transport with no authentication, a
  permissions split that does not hold.
- **A claim that leads someone to believe they are protected when they are not.**
  For example, presenting a read-only agent surface as protection against a sync
  conflict, or presenting a replica as a backup. This is a real harm class, not a
  nitpick: believing the wrong thing is how people lose notes.

## How to report

Open a private report through GitHub's security advisory flow for this repository
(Security tab → Report a vulnerability). If that is unavailable, open a regular
issue that says *only* that you have a security report and would like a private
channel — do not put the detail in the public issue.

Please include:

- The file and the section.
- What the text says, and what actually happens.
- The setup you observed it on, and whether you lost or risked any data.

## What to expect

This is a small documentation project maintained in someone's spare time. There is
no response SLA and it would be dishonest to invent one. Reports that describe a
way to lose data will be handled first.

## Out of scope

Vulnerabilities in the software this guide recommends — Syncthing, Obsidian,
Obsidian Headless, the MCP servers named in
[agent-access.md](docs/agent-access.md). Report those upstream. If a project named
here turns out to be unsafe or abandoned, that is in scope for this repository as
a documentation correction.
