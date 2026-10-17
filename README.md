# Commerant Agent – Releases

Download releases of **Commerant Agent**, the local connector that sends files from your systems to the Commerant platform. It runs on Windows (64-bit) and Linux (amd64 and arm64).

This repository only contains release downloads and customer documentation. The agent itself is developed by Commerant B.V.

## Download

Get the latest version from the [Releases page](../../releases/latest).

| File | Platform | Description |
|---|---|---|
| `commerant-agent.zip` | Windows | Full package: `commerant-agent.exe` and an example configuration (`config.toml.example`). Use this for a first installation. |
| `commerant-agent.exe` | Windows | The executable only. Used by the built-in updater; you can also use it to replace an existing installation manually. |
| `commerant-agent.exe.sha256` | Windows | SHA-256 checksum of `commerant-agent.exe`. |
| `commerant-agent_linux_amd64.tar.gz` | Linux x86-64 | Archive containing one file: `commerant-agent`. |
| `commerant-agent_linux_arm64.tar.gz` | Linux ARM64 | Archive containing one file: `commerant-agent`. |
| `commerant-agent_linux_*.tar.gz.sha256` | Linux | SHA-256 checksum of the matching archive. |

Always download from this repository's Releases page.

### Verify your download (optional)

Compare the checksum of the file you downloaded with the value in its `.sha256` file. The two hashes must be identical (case does not matter). If they differ, do not run the file and contact Commerant.

Windows (PowerShell):

```powershell
(Get-FileHash .\commerant-agent.exe -Algorithm SHA256).Hash
Get-Content .\commerant-agent.exe.sha256
```

If you use `commerant-agent.zip`, extract it first and check the extracted `commerant-agent.exe`.

Linux, in the folder with both files:

```bash
sha256sum -c commerant-agent_linux_amd64.tar.gz.sha256
```

## Requirements

- Windows 64-bit, or Linux (amd64 or arm64) with systemd for running as a service
- Outbound HTTPS access to your Commerant endpoint
- A Commerant API key for your account (provided by Commerant)
- A folder where your ERP or business system writes its export files

## Installation on Windows

1. Download `commerant-agent.zip` and extract it to a permanent folder, for example `C:\CommerantAgent`.
2. Start `commerant-agent.exe`.
3. On first start the agent creates a `config.toml` and opens the **Settings** page of its management page at <http://127.0.0.1:9800/settings> (only reachable from the same machine). If no browser opens, open that address yourself.
4. Enter the watched folder, the Commerant endpoint and your API key. Use **Test connection** to check the endpoint and key, then click **Save**. Changes apply immediately; no restart is needed.

Until an API key is set, the agent sends nothing and the dashboard shows "Setup not finished". Settings are validated when you save: the poll interval must be at least 10 seconds, the watched folder must exist, and the endpoint must be an https address.

Only one agent can run per data folder. A second start, or starting the service while the agent runs in a window, stops with a message instead of sending files twice.

If Windows SmartScreen shows a warning on first start, check that the file was downloaded from this repository before choosing to run it.

### Run as a Windows service

On the **Service** page you can install the agent as a Windows service so it starts automatically. This requires starting the agent as administrator; without administrator rights the page tells you so. The service runs without the management page; to change settings, stop the service and start the agent normally, or edit `config.toml` and restart the service.

## Installation on Linux

The Linux agent runs as a systemd service under its own user. Replace `amd64` with `arm64` on ARM systems.

1. Create the user and folder, and unpack the archive:

   ```bash
   sudo useradd --system --home-dir /opt/commerant-agent --shell /usr/sbin/nologin commerant
   sudo mkdir -p /opt/commerant-agent
   sudo tar -xzf commerant-agent_linux_amd64.tar.gz -C /opt/commerant-agent
   sudo chown -R commerant:commerant /opt/commerant-agent
   ```

2. Create `/opt/commerant-agent/config.toml` (see [Configuration](#configuration)) and restrict it, because it contains your API key:

   ```bash
   sudo chown commerant:commerant /opt/commerant-agent/config.toml
   sudo chmod 600 /opt/commerant-agent/config.toml
   ```

   The `commerant` user must be able to read and write the watched folder.

3. Create `/etc/systemd/system/commerant-agent.service`:

   ```ini
   [Unit]
   Description=Commerant Agent
   After=network-online.target
   Wants=network-online.target

   [Service]
   Type=simple
   WorkingDirectory=/opt/commerant-agent
   ExecStart=/opt/commerant-agent/commerant-agent --service
   Restart=on-failure
   RestartSec=10
   User=commerant
   Group=commerant

   [Install]
   WantedBy=multi-user.target
   ```

4. Start it:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable --now commerant-agent
   systemctl status commerant-agent
   ```

Use `journalctl -u commerant-agent` to see the service output. Windows service management on the **Service** page of the management page works on Windows only.

## Configuration

`config.toml` is stored next to the agent executable. The included `config.toml.example` documents every setting. The main ones:

```toml
[source]
type = "directory"
watch_dir = "C:\\Data\\Inbound"   # Linux example: "/var/lib/commerant/inbound"
file_pattern = ""        # empty = all files, or e.g. "*.json"
processed_dir = "processed"

[destination]
endpoint = "https://<your-commerant-endpoint>/api/v1/inbound"
api_key = ""

[schedule]
interval = "5m"          # at least 10s, for example "30s", "5m" or "1h"

[logging]
level = "info"           # debug, info, warn or error; applies immediately
max_mb = 10              # rotate agent.log at this size
max_age_days = 30        # remove rotated logs older than this
```

- The agent scans `watch_dir` at every interval; subfolders are not scanned.
- Successfully sent files are moved to `processed_dir` (a subfolder of `watch_dir`) and renamed with a `.done` suffix. Files that fail to send stay in place and are retried at the next poll.
- A `.json` file that is not valid JSON can never be sent as it is. It is moved to a `failed` subfolder of `watch_dir` with a `.failed` suffix and is not retried. Fix the file and put it back in the watched folder.
- Files starting with `.` or ending in `.tmp`, `.part`, `.writing` or `.done` are skipped.
- Use a watched folder that contains only the files you want to send.
- Treat `config.toml` as confidential: it contains your API key.

## Logs and troubleshooting

| File | Content |
|---|---|
| `data/logs/agent.log` | Agent log. Rotated at `max_mb`; rotated files are removed after `max_age_days`. |
| `agent-startup-error.log` | Why the agent could not start. Located next to the executable. If the agent cannot start, it stops with an error and writes the reason here. |

Paths are relative to the agent folder (`C:\CommerantAgent` or `/opt/commerant-agent` in the examples above). When the agent runs as a service there is no management page, so read `agent.log` directly. On the **Log** page of a normal run you can view and download the log. On Linux the **Service** page only shows the `systemctl` commands to manage the service.

## Updating

Updates are installed automatically. Version numbers follow the release tags (`vX.Y.Z`). The agent checks this repository for a newer release shortly after it starts and then every 6 hours. When there is one it:

1. downloads the release file and verifies it against the release's `.sha256` file (a missing or mismatching checksum rejects the update);
2. replaces its own executable, keeping the previous one as a backup;
3. restarts itself, as a Windows service, a systemd service or a foreground run.

No action is needed. If the new version does not start properly, the agent returns to the previous version automatically and does not retry that release. The **Update** page shows the current version and can start an update right away.

Requirements: the account the agent runs as must be allowed to write to the installation folder (on Linux, make `/opt/commerant-agent` owned by the `commerant` user: `sudo chown -R commerant:commerant /opt/commerant-agent`).

To turn automatic installation off, or to change how often it checks, edit `config.toml` and restart the agent:

```toml
[update]
enabled = true          # false = never check
auto_install = true     # false = only log that an update is available; install from the Update page
check_interval = "6h"
```

You can still update by hand: stop the agent (for a Windows service, for example `Stop-Service CommerantAgent` in an administrator PowerShell), replace the executable (`commerant-agent.exe`, or `commerant-agent` extracted from the archive) with the one from the latest release, and start it again (`Start-Service CommerantAgent`, or `sudo systemctl start commerant-agent` on Linux). Your `config.toml` is kept.

See the [release notes](../../releases) for what changed in each version.

## Support

See [SUPPORT.md](SUPPORT.md). To report a security issue, see [SECURITY.md](SECURITY.md).

## License

Copyright © Commerant B.V. All rights reserved. The agent is provided to Commerant customers for use with the Commerant platform.
