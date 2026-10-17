# Support

## Customers

Contact your Commerant representative for help with installation, configuration or your API key.

## Questions and bugs about a release

You can open an [issue](../../issues/new/choose) in this repository for questions or problems with a released version. This repository is public, so:

- Never post API keys, passwords, `config.toml` contents or business data.
- Remove customer names and file contents from logs before attaching them.

## What to include

- Agent version: run `commerant-agent.exe -version` (Windows) or `./commerant-agent -version` (Linux)
- Operating system and version (Windows or Linux distribution)
- Whether the agent runs as a service
- What you expected and what happened, with relevant lines from `data/logs/agent.log` (next to the agent; the **Log** page is not available when running as a service)
- If the agent does not start: the contents of `agent-startup-error.log` (next to the executable)

## Security issues

Do not use issues. See [SECURITY.md](SECURITY.md).
