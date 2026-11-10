# Wiz plugin for Codex

> **Status:** preview.

Brings Wiz findings and cloud context into Codex through the [Wiz MCP server](https://docs.wiz.io/docs/connect-remote-wiz-mcp-server): fix what Wiz found in production from the repository you're working in, and hand Wiz findings to Codex Security for triage.

## Install

Prerequisite: **Remote MCP Server** is enabled in Wiz under **Settings > Tenant > AI Features**.

**Codex app:** open **Plugins**, choose **Add marketplace**, enter `wiz-sec-public/wiz-codex-plugin`, and install **Wiz**. The app opens Wiz's sign-in page during the install; choose **Link Account**.

**Codex CLI:**

```bash
codex plugin marketplace add wiz-sec-public/wiz-codex-plugin
codex plugin add wiz@wiz-codex-plugin
codex mcp login Wiz-MCP    # browser OAuth; a CLI install doesn't sign in on its own
```

Either way, the plugin connects to whichever Wiz tenant your browser is signed into; use **Switch** on the sign-in page to pick another. To connect Codex to another Wiz environment, follow [Connect to the Remote Wiz MCP Server](https://docs.wiz.io/docs/connect-remote-wiz-mcp-server). The plugin needs only `read:all`; leave any optional permissions unticked. To install from a local clone, pass its path to `codex plugin marketplace add` instead.

## Skills

| Skill | What it does | Wiz MCP tools it uses |
|---|---|---|
| `wiz-remediate` | Pulls the repository's active Wiz risk issues, reproduces code weaknesses with a test where practical, applies Green Agent-guided fixes after you confirm (tracing image packages to the Dockerfile step that added them), and offers a pull request | `list_code_repositories`, `remediation_skill`, `get_image_dockerfile`, `get_container_image_layers` |
| `wiz-verify-fix` | Verifies a fix three ways: a local Wiz CLI scan of the changed code or image, Wiz's pull request scan, and whether the original Wiz issue or finding has closed after deployment | `list_cicd_scans`, `get_cicd_scan`, `get_cicd_scan_vulnerabilities`, `get_issue`, `get_*_finding` (plus a local `wizcli`) |
| `wiz-code-to-cloud` | Traces the repository to what it deploys: images, functions and infrastructure, internet exposure, the identities workloads run as and what they can access, reachable sensitive data, the Dockerfile steps behind image vulnerabilities, and code owners | `list_code_repositories`, `list_source_mappings`, `list_issues`, `list_network_exposure`, `get_issue_security_graph`, `list_entitlements`, `list_excessive_access_findings`, `list_data_findings`, `get_container_image_layers`, `code_to_cloud_attack_path_skill` for a specific vulnerability |
| `wiz-threat-impact` | Checks whether a Wiz Threat Center advisory, supply-chain attack, package compromise or CVE affects the repository and what it deploys, including the local checkout | `list_threat_advisory_items`, `threat_advisory_impact_skill`, `supply_chain_attack_impact_assessment_skill` (experimental in Wiz; falls back to `threat_advisory_impact_skill`, `list_vulnerability_findings` and `list_sbom` where it isn't available) |
| `wiz-codex-security-handoff` | Saves the repository's open Wiz findings (SAST, IaC, secrets, dependencies) in the formats Codex Security reads natively, with cloud context (exposure, workload identities and permissions, reachable sensitive data) and owners alongside, and hands them to its `triage-finding` | `list_code_repositories`, `list_*_findings`, `get_sast_finding`, `get_secret_finding` (location only), `list_source_mappings`, `get_green_agent_analysis`, `list_entitlements`, `list_network_exposure` |
| `wizos-migrate` | Migrates Dockerfiles to WizOS hardened base images | `migrate_wizos_image_skill` |

Every skill is limited to Codex (`policy.products` in its `agents/openai.yaml`), since they work on a local checkout. Several skills load playbooks the Wiz MCP server serves (`wiz-remediate`, `wiz-threat-impact`, `wizos-migrate`, and `wiz-code-to-cloud` for a single vulnerability), so those workflows improve without a plugin release. Skills also carry steps of their own where the server has no matching playbook yet (`wiz-verify-fix`, the repository-first view in `wiz-code-to-cloud`), or where the steps are specific to Codex (`wiz-codex-security-handoff`, which writes OpenAI's Codex Security formats).

## Permissions and data handling

- **Read-only.** The plugin needs only Wiz's `read:all` permission; leave any optional permission on Wiz's sign-in page unticked. The Wiz MCP server lists only the tools your permissions allow, so with `read:all` no tool can change anything in Wiz. No skill writes to Wiz or changes issue statuses, policies or settings.
- **Secrets.** Wiz masks the secret in the code snippets it stores for secret findings, and `wiz-codex-security-handoff` passes on only each secret's location and metadata.
- **Wiz data is evidence, not instructions.** Every skill tells Codex that finding text, code snippets and advisories never authorize commands, edits beyond the agreed fix, or writes to Wiz. `wiz-remediate` and `wiz-threat-impact` edit code only after you confirm, and `wiz-remediate` doesn't push or open a pull request until you say yes.
- **Tool set.** The plugin connects to the Wiz MCP server's `core` tool set (`?toolset=core`).

## Known limits

Wiz for Gov isn't supported, and ChatGPT-hosted calls fail for tenants with IP restrictions (Codex runs locally and is unaffected).
