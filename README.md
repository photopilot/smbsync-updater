# smbsync-updater

🇬🇧 English — [Nederlandse versie](README.nl.md)

This repository hosts the up-to-date **smbsync** binaries. smbsync watches an
SMB share (e.g. a StyleShoots machine) and syncs photo folders to Photopilot.

## Installation

1. Download the binary for your platform from `smbsync/<version>/`:
   - `windows-amd64.exe.gz` — Windows 64-bit (most common)
   - `windows-386.exe.gz` — Windows 32-bit
   - `darwin-arm64.gz` / `darwin-amd64.gz` — macOS (Apple Silicon / Intel)
   - `linux-amd64.gz` / `linux-arm64.gz` — Linux
2. Unpack the `.gz` file (on Windows use 7-Zip, on macOS/Linux run `gunzip <file>`).
3. Place the binary in its own folder and create a `.env` file next to it:

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

4. Start the binary. It keeps running and checks the share every 15 seconds.

## How folders are synced

- **Shoots** — folders named `S-<set id>` (e.g. `S-1234`) are uploaded as a
  shoot for that set.
- **Packshots** — every other folder is treated as an article identifier: a
  scanned **EAN/barcode** (e.g. `8712345678906`) or a **public article ID**
  (e.g. `12345-001`, an optional `P-` prefix is stripped). The article must
  already exist in Photopilot; unknown folders are skipped and retried every
  15 minutes. Within a packshot folder, images with `back`, `side` or `label`
  in the filename are attached as that view; every other image becomes the
  front image of its own packshot job.

A folder is only synced after it has been untouched for 10 minutes, so the
photographer can finish shooting first. After a successful sync the files and
the folder are removed from the share (folders containing RAW `.cr2` or video
files are left in place).

## Error monitoring

Errors are reported to Sentry. Set `SENTRY_DSN` in the `.env` file to override
the default DSN.

## Releasing a new version

From the [smbsync](https://github.com/photopilot/smbsync) repository:

```bash
./update-version.sh   # bump the version number in main.go
./release.sh          # build all platforms into ../smbsync-updater
cd ../smbsync-updater && git add . && git commit -m "Update binaries" && git push
```
