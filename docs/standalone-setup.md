# Independently configured remote MCP servers

No Docker is required. Configure two independent hosted MCP endpoints: GitHub for issue and deployment evidence, and Exa for web search and documentation retrieval.

## Configure Codex

Merge [config/codex-mcp.toml](../config/codex-mcp.toml) into `~/.codex/config.toml`, preserving unrelated settings. Replace the old GitHub definition and remove `exa_standalone` if present rather than adding duplicate tables.

GitHub uses `https://api.githubcopilot.com/mcp/`, with read-only tools and a personal access token supplied through `GITHUB_PERSONAL_ACCESS_TOKEN`. Create a short-lived token with only the read access needed for the repositories being investigated. SSH keys used for Git pushes do not authenticate API requests.

Enter the token locally in zsh, without echoing it or adding its value to shell history:

```sh
read -rs 'GITHUB_PERSONAL_ACCESS_TOKEN?GitHub PAT: '
printf '\n'
export GITHUB_PERSONAL_ACCESS_TOKEN
codex mcp list
codex
```

Launch a fresh Codex CLI session from this terminal in the project directory. An already running desktop app or IDE may not inherit the terminal environment. Never paste the token into chat or commit it.

## Exa hosted server

`exa_standalone` connects to `https://mcp.exa.ai/mcp`. Exa documents basic hosted access without an API key. No GitHub credentials are forwarded to Exa. Available tools and usage limits are controlled by the provider; discover the tools in the fresh session before selecting them.

Use Exa to find authoritative documentation and retrieve the relevant content. Search results alone are not proof of a root cause. Prefer original GitHub documentation and distinguish source claims from inference.

## Verify the workflow

After both servers are available, start a fresh session and ask:

> Use only github_standalone and exa_standalone for this test. Investigate GitHub Pages reporting “Deployment failed, try again later” during deployment-status polling. Search existing issues in actions/deploy-pages, retrieve GitHub's custom Pages workflow documentation, and recommend the next action. Record actual tool names, arguments, URLs, returned status and failures. Do not modify external systems or substitute built-in integrations.

Require successful issue search through GitHub and documentation search and retrieval through Exa. A configuration listing alone does not prove runtime connectivity. Preserve the existing evidence as historical built-in test results and record the new test separately. Neither remote server has completed a runtime test in this session.

## Authoritative references

- [GitHub hosted MCP setup for Codex](https://github.com/github/github-mcp-server/blob/main/docs/installation-guides/install-codex.md)
- [GitHub remote read-only headers](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md)
- [Exa hosted MCP setup](https://exa.ai/mcp)
- [Codex MCP configuration](https://developers.openai.com/codex/mcp/)
