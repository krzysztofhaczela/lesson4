# MCP-enabled DevOps agent homework

Investigate deployment failures using reusable agent rules, an investigation skill, and two independently configured remote MCP servers.

## MCP servers

| Server | Endpoint | Purpose | Authentication |
| --- | --- | --- | --- |
| `github_standalone` | `https://api.githubcopilot.com/mcp/` | Search issues and inspect repository and Actions evidence | GitHub PAT in the local, Git-ignored `.mcp.json` |
| `exa_standalone` | `https://mcp.exa.ai/mcp` | Search the web and retrieve authoritative documentation | Basic hosted access requires no API key |

No Docker is required. Follow [the setup guide](docs/standalone-setup.md). Copy [.mcp.json.example](.mcp.json.example) to `.mcp.json` and replace `<yourtokenhere>` with your GitHub token; `.mcp.json` is Git-ignored and must never be committed. Git SSH authentication is separate from GitHub API authentication.

Configuration is prepared. A fresh investigation through both configured servers is pending; no successful runtime test is claimed. Listing configured servers alone does not verify connectivity.

## Run the workflow

Start a fresh Claude Code session (`claude`) in this directory with both MCP servers approved and your token filled in to `.mcp.json`. Give it a natural-language goal, for example:

> Investigate this deployment problem: GitHub Pages reports “Deployment failed, try again later” while polling deployment status. Check whether it is already tracked in actions/deploy-pages, find relevant technical information, and recommend the next action.

This example is a public demonstration. For an actual incident, provide the repository, failing run URL and error. The prompt needs no tool names or tool-call syntax.

`CLAUDE.md` imports `AGENTS.md`, which routes the request to `skills/devops-investigate/SKILL.md`. The workflow searches GitHub tracking, retrieves authoritative information through Exa, and correlates the evidence into a recommendation. It remains read-only unless changes are explicitly authorized.

## Project files

- [AGENTS.md](AGENTS.md): MCP usage rules, server selection and skill routing.
- [Investigation skill](skills/devops-investigate/SKILL.md): reusable investigation workflow.
- [CLAUDE.md](CLAUDE.md): imports `AGENTS.md` so Claude Code loads the rules.
- [MCP configuration template](.mcp.json.example): credential-free Claude Code server definitions; copy to `.mcp.json` and add your token.
- [Setup guide](docs/standalone-setup.md): authentication and fresh-session verification.
- [Investigation report](evidence/investigation.md): cleared for the next investigation.
- [Tool results](evidence/tool-results.json): empty call list for the next investigation.

Record actual tool names, arguments, concise returned evidence, source URLs and failures after the next run. Do not substitute built-in app integrations for the two configured servers or claim completion until both return useful evidence.
