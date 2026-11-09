# MCP-enabled DevOps agent homework

This project contains reusable agent rules, an investigation skill, and evidence from a live two-integration test on 9 November 2026.

## Connections used

| MCP integration exposed to this agent | Discovered tool | Successful use |
| --- | --- | --- |
| GitHub (`mcp__codex_apps__github`) | `github_search_issues` | Searched real deployment issues |
| Search Service (`mcp__codex_apps__search_service`) | `web_run` | Retrieved authoritative GitHub documentation |

These connections were already supplied by the Codex desktop environment. This work reused and verified them; it did not install two new servers or change account configuration. Their tool namespaces are distinct, although both are managed through the app. No tokens are stored in this project.

`codex mcp list` also ran successfully, with a warning about PATH alias creation. It listed local app/runtime servers, not these app-provided integrations; that listing alone is not evidence of the two connections. The successful calls in `evidence/tool-results.json` are the connection evidence. If the assessor requires two separately configured standalone MCP endpoints, that additional setup and a new-session test remain necessary.

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

Workflow: natural-language goal → AGENTS.md → investigation skill → GitHub issue search → Search Service documentation retrieval → evidence-based recommendation.
