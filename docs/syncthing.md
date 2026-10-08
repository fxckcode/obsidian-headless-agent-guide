# Self-hosted transport: Syncthing, one-way

No subscription, no plugin, no desktop app on the server. Syncthing syncs plain
files between devices, and its **receive only** folder type is the feature that
makes this a good fit for an agent-facing replica.

Sources: [Syncthing folder types](https://docs.syncthing.net/users/foldertypes.html)

## The shape

```text
PC (Obsidian, send only)  ──►  server (receive only)  ──►  MCP server (read only)
```

Both ends are deliberately one-directional, for the same reason: nothing on the
server should ever be able to change what is on the PC.

## What receive only actually does

Read this before you rely on it, because the behaviour is counter-intuitive.

From Syncthing's documentation: receive only is intended for *"replication
mirrors, backup destinations, or other cases where no local modifications are
expected or allowed"*.

- Local changes are **preserved** and are **not** distributed to other devices.
- Local modifications do **not** make the folder look out of sync **on this
  device**. It is the *other* devices that see this one as behind. Looking at the
  wrong dashboard tells you the wrong thing.
- Syncthing shows a red **Revert Local Changes** button. Using it deletes files
  that were added locally and re-syncs modified or deleted ones from the cluster.
- If a locally modified file is later modified by the cluster, that is a **sync
  conflict**, resolved with the cluster version winning. The conflict copy is not
  sent anywhere, and it is deleted if you use Revert Local Changes.

And on the sending side, **send only** means changes from other devices are
ignored — the folder can go "out of sync" and Syncthing offers a red **Override
Changes** button. Be careful with that one: it enforces *this host's* state onto
the rest of the cluster, overwriting and deleting as needed.

## Server side

Install Syncthing, or run it in Docker if the host has no root access (this is
often the reason to choose the container: no package manager, no sudo).

**Keep the Web UI off the network.** Bind it to loopback:

```bash
-p 127.0.0.1:8384:8384     # Web UI and API: this machine only
-p 22000:22000/tcp         # sync
-p 22000:22000/udp
-p 21027:21027/udp         # local discovery
```

Verify that claim rather than trusting it. From the host:

```bash
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8384/rest/noauth/health   # 200
curl -s -o /dev/null -w '%{http_code}\n' http://<lan-ip>:8384/                      # 000
```

`000` from the LAN is the result you want. If you need the UI from your laptop,
tunnel it over SSH or a VPN — do not publish the port, and do not put it behind a
reverse proxy on the same host.

## Pre-create the folder as receive only

You can either accept the folder the PC offers, or create it first on the server
and tell the PC to use that Folder ID. Creating it first removes the pending-folder
dance and makes the mode explicit:

Set **Folder ID** to a fixed string of your choosing, point the path at the vault
directory, and set the type to `receiveonly`.

## PC side

1. Install Syncthing.
2. Add the server as a remote device, using the **device ID** the server prints
   (`syncthing --device-id`, or the `myID` field of `/rest/system/status`).
3. Share the vault folder, using the same **Folder ID** you created on the
   server, folder type **Send Only**, and tick the server device under Sharing.
4. Give the server the PC's device ID so it can accept it.

## What was verified, and what was not

Being precise about this, because it is the difference between a guide and a blog
post:

**Verified on a real host by the author of this guide:**

- Syncthing runs in Docker on a host with no passwordless sudo.
- The folder is created as `receiveonly` before the PC ever connects.
- The Web UI/API answers `200` on loopback and `000` from the LAN with the port
  published as shown above.
- The replica starts empty and the vault directory is a real directory Syncthing
  owns.

**Not verified:** an actual file transfer between two devices. Both halves were
stood up and the receiving side confirmed listening and configured, but no second
device completed a sync in that environment. If you are reading this to decide
whether to trust Syncthing, that is a gap you should close yourself — the
verification procedure to do it is in [verification.md](verification.md).

## Do not layer sync services

If you also run Obsidian Sync on the PC, you now have two systems writing to the
same directory. Obsidian's FAQ is explicit:

> We do not recommend using a third-party sync service alongside Obsidian Sync.
> Using a third-party service and Obsidian Sync on multiple devices can lead to
> conflicts, resulting in duplicate or corrupted files.

Self-hosted LiveSync says it more directly: it is *not compatible* with Obsidian
Sync and cannot synchronise with it, and its own instructions say not to enable it
alongside another synchronisation solution.

**There is a data-loss case hiding in that paragraph.** Obsidian's FAQ goes on to
warn that cloud clients offering "Files on-demand" or "Online-only files"
(OneDrive, Dropbox) download files only when needed and remove them locally to
save space. Because those files are not present locally, **Obsidian Sync
interprets them as deleted and removes them from your remote vault.** If any part
of your stack uses on-demand files, disable it before you go further.

## Alternative: Self-hosted LiveSync + CouchDB

If you need mobile clients or real-time sync, [Self-hosted
LiveSync](https://github.com/vrtmrz/obsidian-livesync) with a CouchDB backend is
the community answer, with the plugin handling conflict resolution. It costs you a
database and a plugin on every device. It is a heavier answer than Syncthing, and
a better one if mobile matters.

## Next

- Let an agent read the vault: [agent-access.md](agent-access.md)
- Prove the copy really arrived: [verification.md](verification.md)
