# MCP-enabled DevOps agent homework

This project contains reusable agent rules, an investigation skill, and a standalone two-server setup. The original evidence is from a built-in integration test on 9 November 2026; it does not prove standalone setup.

## Standalone servers (required setup)

Follow [the setup guide](docs/standalone-setup.md) for URL-based MCP configuration without Docker. [Codex configuration](config/codex-mcp.toml) connects to GitHub's hosted MCP server using a personal access token from the environment. Exa connects to its hosted MCP URL for search and documentation retrieval; its basic setup requires no API key. Your Git SSH key is separate from API authentication.

Status: GitHub and Exa remote definitions prepared; a fresh two-server runtime test is pending. The evidence below remains historical until the new test succeeds.

## Historical connections used (built-in test only)

| MCP integration exposed to this agent | Discovered tool | Successful use |
| --- | --- | --- |
| GitHub (`mcp__codex_apps__github`) | `github_search_issues` | Searched real deployment issues |
| Search Service (`mcp__codex_apps__search_service`) | `web_run` | Retrieved authoritative GitHub documentation |

These connections were already supplied by the Codex desktop environment. This work reused and verified them; it did not install two new servers or change account configuration. Their tool namespaces are distinct, although both are managed through the app. No tokens are stored in this project.

`codex mcp list` also ran successfully, with a warning about PATH alias creation. It listed local app/runtime servers, not these app-provided integrations; that listing alone is not evidence of the two connections. The successful calls in `evidence/tool-results.json` are the connection evidence. If the assessor requires two separately configured standalone MCP endpoints, the remote definitions are now provided; a new-session runtime test remains necessary.

## Run the workflow

Open this project in Codex with both integrations available. Give it a goal, for example:

> Investigate this deployment problem: GitHub Pages reports “Deployment failed, try again later” while polling deployment status. Check whether it is already tracked in actions/deploy-pages, find relevant technical information, and recommend the next action.

For your own incident, replace the example with the repository, failing run URL and error. The prompt needs no tool names or tool-call syntax.

`AGENTS.md` routes deployment investigations to `skills/devops-investigate/SKILL.md`. This explicit project routing makes the requested skill location usable without relying on automatic skill-directory discovery. This session read both files before executing the recorded investigation. A fresh-session test of automatic routing has not been performed.

## Deliverables

- `AGENTS.md`: five assignment rules plus four additional rules and skill routing.
- `skills/devops-investigate/SKILL.md`: short, reusable multi-server investigation workflow.
- `evidence/investigation.md`: test goal, tool selection, findings, correlation and next action.
- `evidence/tool-results.json`: exact investigation arguments and condensed returned evidence, not a verbatim server transcript.

Workflow: natural-language goal → AGENTS.md → investigation skill → GitHub issue search → Exa documentation search and retrieval → evidence-based recommendation.
