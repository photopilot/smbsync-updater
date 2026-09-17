# smbsync-updater

🇬🇧 English — [Nederlandse versie](README.nl.md)

This repository hosts the up-to-date **smbsync** binaries. smbsync watches an
SMB share (e.g. a StyleShoots machine) and syncs photo folders to Photopilot.

## Installation

> Still running **v25 or older**? Those versions do not update themselves.
> Install the latest version manually once, using the steps below; after that
> it takes care of itself.

1. Download the binary for your platform from the
   [**latest release**](https://github.com/photopilot/smbsync-updater/releases/latest)
   (not from the `smbsync/` folder in this repo — that stops at v25):
   - `windows-amd64.exe` — Windows 64-bit (most common, signed)
   - `windows-386.exe` — Windows 32-bit
   - `darwin-arm64` / `darwin-amd64` — macOS (Apple Silicon / Intel)
   - `linux-amd64` / `linux-arm64` — Linux
2. Place the binary in **its own folder** (e.g. `C:\smbsync\`) and rename it
   to `smbsync.exe`. The folder must be writable for the account smbsync runs
   as: the log file and the self-update write there.
3. Create a `.env` file in the **same folder**:

```
CUSTOMER=Company Name
SMB_IP=00.00.00.00
SMB_USERNAME=styleshootssharing
SMB_PASSWORD=
SMB_MOUNT=StyleshootsDrive
START_DIR=test
API_TOKEN=
API_URL=
LOG_LEVEL=info
TZ=Europe/Amsterdam
```

   **`CUSTOMER` is required and must be unique per installation** (e.g.
   `Van Dijk`, or `Van Dijk studio 2` with several machines). The installation
   shows up in monitoring under that name; two installations sharing a name
   hide each other's outages.

4. Start smbsync. It keeps running and checks the share every 15 seconds.

### Windows: as a scheduled task

This keeps smbsync running after a server reboot, even with nobody logged in:

1. Task Scheduler → **Create Task**.
2. *General*: "Run whether user is logged on or not".
3. *Triggers*: "At startup" only. Do **not** add a repeating trigger: after a
   self-update smbsync keeps running detached from the task, and a repetition
   would start a second instance next to it (duplicate uploads).
4. *Actions*: program `C:\smbsync\smbsync.exe`, "Start in" `C:\smbsync`.
5. *Settings*: "If the task is already running: **do not start a new
   instance**", and **untick** "Stop the task if it runs longer than".

### Checking that it works

- `smbsync.log` appears in the folder. The first lines should be
  `Starting smbsync currentVersion=<number>` and `Connected to SMB share`.
- Tell Photopilot the `CUSTOMER` name you chose; we will see the installation
  in our monitoring within 5 minutes.
- Not working? Send us `smbsync.log` — it contains the exact error (wrong
  password, share unreachable, `.env` not found, …).

### Network

smbsync needs outbound HTTPS (port 443) to the Photopilot API (`API_URL`),
`api.github.com` and `github.com` (updates) and `*.ingest.de.sentry.io`
(monitoring), plus port 445 to the SMB share.

## Updates

From v26 on smbsync updates itself: every 30 minutes it checks the latest
release here, verifies the SHA256 and restarts between two sync rounds — never
in the middle of an upload. Set `SELF_UPDATE=false` in `.env` to turn it off.

## How folders are synced

- **Shoots** — folders named `S-<set id>` (e.g. `S-1234`) are uploaded as a
  shoot for that set.
- **Packshots** — every other folder is treated as an article identifier: a
  scanned **EAN/barcode** (e.g. `8712345678906`) or a **public article ID**
  (e.g. `12345-001`). The article must already exist in Photopilot; unknown
  folders are skipped and retried every 15 minutes. Within a packshot folder,
  images with `back`, `side` or `label` in the filename are attached as that
  view; every other image becomes the front image of its own packshot job.

A folder is only synced after it has been untouched for 10 minutes, so the
photographer can finish shooting first. After a successful sync the files and
the folder are removed from the share (folders containing RAW `.cr2` or video
files are left in place).

## Monitoring

- **Heartbeat** — every installation checks in with Photopilot every 5 minutes
  (Sentry Crons, monitor `smbsync-<customer>`). If that stops for ~15 minutes,
  or the share is unreachable for that long, Photopilot is alerted.
- **Logs** — log lines go to `smbsync.log` next to the executable (10 MB max,
  then `smbsync.log.1`) and to Photopilot, so we can look along without access
  to the server. `LOG_FILE=<path>` changes the location, `LOG_FILE=off`
  disables the file.
- **Errors** — errors and crashes are reported to Sentry. Set `SENTRY_DSN` in
  `.env` to override the default DSN.
- A lost share does not end smbsync: it reconnects every 30 seconds.

## Releasing a new version

Releases are built, signed and published here automatically whenever a change
to `smbsync/` lands on `main` of the Photopilot monorepo. The version number is
the latest release + 1. The `smbsync/` folder in this repo only holds the old
versions up to v25.
