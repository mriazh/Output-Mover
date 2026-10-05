# Output Mover

Automated daily output mover for office automation projects (`AP-Report-Auto`, `Network-Daily-Report-Automation`, `MRTG-TelkomCare-Report-Automation`, `Automated-MRTG-to-Excel-Report`). Syncs local generated outputs to Google Drive root (`NETWORK`) via `rclone`, cleans up old archives, renames files for standard archiving, and posts a success/failure notification.

Runs on the office Debian PC under a systemd timer. Everything here works offline except the `rclone sync` call, which pushes finalized outputs to cloud storage.

## What a run does

1. Locate generated outputs across the configured local project directories.
2. Validate and stage files into structured sync directories.
3. Execute `rclone sync` to push files securely to Google Drive root (`NETWORK`).
4. Log transfer status and clean up temporary or expired local staging cache.
5. Send status notices to the configured notification webhook if enabled.

A failed sync or missing source directory preserves existing remote targets and leaves local outputs intact until the next successful run.

## Layout

```
config/.env                 local secrets and settings (git-ignored)
src/output_mover/           the package
deploy/systemd/             service + timer units
output/                     local staging and sync logs
docs/                       project specifications and design documents
```

## Requirements

- Debian with Python 3.11+
- `rclone` configured with a valid remote for Google Drive root (`NETWORK`)

## Install (Debian)

The repo lives under the operator's own home directory, matching the other
GitHub projects on this PC, rather than under a system directory. Clone into
`/home/mriazh/Github-PC/Output-Mover` as the `mriazh` user:

```sh
sudo -u mriazh mkdir -p /home/mriazh/Github-PC
sudo -u mriazh git clone <repo> /home/mriazh/Github-PC/Output-Mover
sudo -u mriazh python3 -m venv /home/mriazh/Github-PC/Output-Mover/.venv
sudo -u mriazh /home/mriazh/Github-PC/Output-Mover/.venv/bin/pip install --upgrade pip
```

Copy the example env and fill it in locally. It is never committed:

```sh
sudo -u mriazh cp /home/mriazh/Github-PC/Output-Mover/config/.env.example /home/mriazh/Github-PC/Output-Mover/config/.env
sudo -u mriazh chmod 600 /home/mriazh/Github-PC/Output-Mover/config/.env
$EDITOR /home/mriazh/Github-PC/Output-Mover/config/.env
```

## Schedule

```sh
sudo cp deploy/systemd/output-mover.service /etc/systemd/system/
sudo cp deploy/systemd/output-mover.timer /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now output-mover.timer
systemctl list-timers output-mover.timer
```
