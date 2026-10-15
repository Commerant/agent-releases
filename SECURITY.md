# Security Policy

## Reporting a vulnerability

Please report suspected vulnerabilities privately, not in public issues.

Use [GitHub's private vulnerability reporting](../../security/advisories/new) for this repository, or contact your Commerant representative.

Include the agent version, a description of the issue and steps to reproduce it. Do not include real customer data or credentials.

## Supported versions

Security fixes are released in the latest version. Please update to the [latest release](../../releases/latest) before reporting.

## Handling your credentials

- Your API key is stored in `config.toml` next to the agent. Restrict access to that folder to the people and service accounts that need it.
- If you suspect a key was exposed, ask Commerant to revoke it and issue a new one.
- Download the agent only from this repository's [Releases page](../../releases).
