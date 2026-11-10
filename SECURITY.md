# Security Policy

## Scope

In scope:
- Bugs in this plugin's skills and manifests.
- Supply-chain or prompt-injection risk against this plugin.

Out of scope (report via the Wiz HackerOne program instead):
- The Wiz Cloud platform, Wiz MCP server, or Wiz CLI itself.
- The Wiz tenant authentication flow.

## Reporting

Email `security@wiz.io` with a description, reproduction steps, and impact.

## Coordinated disclosure

Please coordinate public disclosure with our security team before publishing details.

## Verifying authenticity

The plugin's only official source is `https://github.com/wiz-sec-public/wiz-codex-plugin`. Install it only from there, and check that the `repository` field in `.codex-plugin/plugin.json` points to the same place.
