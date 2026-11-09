# Independently configured remote MCP servers

No Docker is required. Configure two independent hosted MCP endpoints: GitHub for issue and deployment evidence, and Exa for web search and documentation retrieval.

## Configure Claude Code

Both servers are defined in the project-scoped `.mcp.json`, which Claude Code loads when started in this directory. That file holds your GitHub token, so it is excluded from Git by `.gitignore`. The repository ships a credential-free template, [.mcp.json.example](../.mcp.json.example).

GitHub uses `https://api.githubcopilot.com/mcp/`, with read-only tools (`X-MCP-Readonly: true`) limited to the `issues`, `repos` and `actions` toolsets, and authenticates with a GitHub personal access token. Create a short-lived token with only the read access needed for the repositories being investigated. SSH keys used for Git pushes do not authenticate API requests.

1. Copy the template:

   ```sh
   cp .mcp.json.example .mcp.json
   ```

2. Open `.mcp.json` and replace `<yourtokenhere>` with your token. Keep the `Bearer ` prefix and the single space, for example `"Authorization": "Bearer github_pat_..."`. A missing prefix or an empty token makes the server reject the connection with `Authorization header is badly formatted`.

3. Confirm the file is ignored before doing anything else with Git:

   ```sh
   git check-ignore .mcp.json
   ```

4. Start Claude Code in the project directory:

   ```sh
   claude mcp list
   claude
   ```

On first start Claude Code asks you to approve the project-scoped servers in `.mcp.json`; approve both. Inside a session, `/mcp` shows each server's status and tools. Servers connect only at session start, so restart Claude Code (or the IDE) after editing `.mcp.json`. Never paste the token into chat or commit `.mcp.json`; if the token is exposed, revoke it on GitHub and create a new one.

## Exa hosted server

`exa_standalone` connects to `https://mcp.exa.ai/mcp`. Exa documents basic hosted access without an API key. No GitHub credentials are forwarded to Exa. Available tools and usage limits are controlled by the provider; discover the tools in the fresh session before selecting them.

Use Exa to find authoritative documentation and retrieve the relevant content. Search results alone are not proof of a root cause. Prefer original GitHub documentation and distinguish source claims from inference.

## Verify the workflow

After both servers are available, start a fresh session and ask:

> Use only github_standalone and exa_standalone for this test. Investigate GitHub Pages reporting “Deployment failed, try again later” during deployment-status polling. Search existing issues in actions/deploy-pages, retrieve GitHub's custom Pages workflow documentation, and recommend the next action. Record actual tool names, arguments, URLs, returned status and failures. Do not modify external systems or substitute built-in integrations.

In Claude Code the tools appear as `mcp__github_standalone__*` and `mcp__exa_standalone__*`. Require successful issue search through GitHub and documentation search and retrieval through Exa. A configuration listing alone does not prove runtime connectivity. Preserve the existing evidence as historical test results and record the new test separately.

## Authoritative references

- [GitHub hosted MCP setup for Claude](https://github.com/github/github-mcp-server/blob/main/docs/installation-guides/install-claude.md)
- [GitHub remote read-only headers](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md)
- [Exa hosted MCP setup](https://exa.ai/mcp)
- [Claude Code MCP configuration](https://code.claude.com/docs/en/mcp)
