# Commerant Agent – Releases

Download releases of **Commerant Agent**, the local connector that sends files from your systems to the Commerant platform.

This repository only contains release downloads and customer documentation. The agent itself is developed by Commerant B.V.

## Download

Get the latest version from the [Releases page](../../releases/latest).

| File | Description |
|---|---|
| `commerant-agent.zip` | Full package: `commerant-agent.exe` and an example configuration (`config.toml.example`). Use this for a first installation. |
| `commerant-agent.exe` | The executable only. Used by the built-in updater; you can also use it to replace an existing installation manually. |

Always download from this repository's Releases page.

## Requirements

- Windows, 64-bit
- Outbound HTTPS access to your Commerant endpoint
- A Commerant API key for your account (provided by Commerant)
- A folder where your ERP or business system writes its export files

## Installation

1. Download `commerant-agent.zip` and extract it to a permanent folder, for example `C:\CommerantAgent`.
2. Start `commerant-agent.exe`.
3. On first start the agent creates a `config.toml` and opens its management page at <http://127.0.0.1:9800> (only reachable from the same machine).
4. Open **Settings**, enter the watched folder, the Commerant endpoint and your API key, and save.

If Windows SmartScreen shows a warning on first start, check that the file was downloaded from this repository before choosing to run it.

### Run as a Windows service

On the **Service** page you can install the agent as a Windows service so it starts automatically. This requires starting the agent as administrator. The service runs without the management page; to change settings, stop the service and start the agent normally, or edit `config.toml` and restart the service.

## Configuration

`config.toml` is stored next to `commerant-agent.exe`. The included `config.toml.example` documents every setting. The main ones:

```toml
[source]
type = "directory"
watch_dir = "C:\\Data\\Inbound"
file_pattern = ""        # empty = all files, or e.g. "*.json"
processed_dir = "processed"

[destination]
endpoint = "https://<your-commerant-endpoint>/api/v1/inbound"
api_key = ""

[schedule]
interval = "5m"
```

- The agent scans `watch_dir` at every interval; subfolders are not scanned.
- Successfully sent files are moved to `processed_dir` (a subfolder of `watch_dir`) and renamed with a `.done` suffix. Failed files stay in place and are retried.
- Files starting with `.` or ending in `.tmp`, `.part`, `.writing` or `.done` are skipped.
- Use a watched folder that contains only the files you want to send.
- Treat `config.toml` as confidential: it contains your API key.

## Updating

Version numbers follow the release tags (`vX.Y.Z`). The agent checks this repository for newer releases and the **Update** page can install one. Restart the agent afterwards to run the new version.

To update manually, stop the agent, replace `commerant-agent.exe` with the one from the latest release, and start it again. Your `config.toml` is kept.

See the [release notes](../../releases) for what changed in each version.

## Support

See [SUPPORT.md](SUPPORT.md). To report a security issue, see [SECURITY.md](SECURITY.md).

## License

Copyright © Commerant B.V. All rights reserved. The agent is provided to Commerant customers for use with the Commerant platform.
