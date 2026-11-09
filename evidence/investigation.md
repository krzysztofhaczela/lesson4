Goal: Verify GitHub MCP connectivity.

Current user-requested recheck (2026-11-09): Runtime ALL_TOOLS inventory again exposes zero github_standalone tools and 89 built-in GitHub connector tools. Consulted docs/standalone-setup.md. No standalone read-only call could be attempted, so connectivity remains unverified; no authentication error was observed. No new external MCP calls were made, and tool-results.json is unchanged. Startup/configuration loading or environment inheritance remain hypotheses. Next verification step: restart the Codex extension with the configured credential available, start a fresh session, discover github_standalone tools, and perform a read-only call.

Latest retry after credential configuration change (2026-11-09): Current runtime ALL_TOOLS inventory contains zero github_standalone tools. No read-only server call can be attempted through that server in this session. No credentials were inspected, no authentication response was observed, and no new MCP server calls were made. Existing tool-results.json calls are preserved. Connectivity remains unverified; next step is a fresh Codex session after restarting the extension, followed by standalone tool discovery and a read-only call. Changes to config/codex-mcp.toml alone affect the example file, not an active Codex configuration.

Environment diagnostic (2026-11-09): Project TOML parsed successfully; github_standalone is present and enabled, points to https://api.githubcopilot.com/mcp/, and references GITHUB_PERSONAL_ACCESS_TOKEN. A boolean-only environment check found that variable absent or empty in the tool subprocess. This does not directly inspect the IDE extension host environment. `codex mcp list` lists github_standalone as enabled with Bearer token authentication, but emits a PATH-alias permission warning; that listing does not test connectivity. Runtime discovery still exposes no github_standalone tools. Hypothesis: the Codex host did not inherit the token. Recommended verification: fully restart the IDE from an environment with the exported token, then start a fresh Codex session and perform a standalone read-only call. No GitHub authentication response has been observed and no new MCP server calls were made during this diagnostic.

Latest recheck (2026-11-09): Runtime ALL_TOOLS discovery again returned zero github_standalone tools and 89 built-in GitHub connector tools. Read docs/standalone-setup.md and confirmed origin is git@github.com:krzysztofhaczela/lesson4.git. No standalone GitHub call was possible, and no authentication failure was observed. Existing recorded calls are historical and were preserved; no new MCP server calls were made. Connectivity remains unverified. Follow the setup document, restart the Codex session with the token available, and retry a read-only standalone repository call.

Scope: Read-only discovery of runtime tools and local setup instructions on 2026-11-09. No external writes.

Findings: Runtime tool discovery exposes no tools from github_standalone. It exposes 89 built-in GitHub connector tools, which repository instructions prohibit substituting for the standalone server. Exa standalone search and fetch tools are exposed but were not called for this GitHub-only check. The local .codex/config.toml defines github_standalone with the hosted endpoint, a bearer-token environment variable, and read-only headers. Configuration alone does not prove connectivity.

Sources: .codex/config.toml; docs/standalone-setup.md; runtime ALL_TOOLS inventory. Setup reference: https://github.com/github/github-mcp-server/blob/main/docs/installation-guides/install-codex.md (not retrieved).

Hypotheses: The current session may not have loaded the configuration or inherited the token environment variable. Neither cause is established by the available evidence.

Missing context: Startup/authentication diagnostics for github_standalone and a successful standalone GitHub tool call. No standalone GitHub call could be attempted because its tools are unavailable; no authentication failure was observed.

Recommended verification: Follow docs/standalone-setup.md to supply the token locally, check the MCP listing, and launch a fresh session from that environment. Then discover github_standalone tools and perform a read-only repository or issue search. This exercise is not verified.

VS Code follow-up: User clarified they use the Codex VS Code extension. Successful Exa standalone search and retrieval of https://developers.openai.com/codex/extend/mcp confirms the extension shares Codex configuration, supports trusted-project .codex/config.toml, and offers gear menu > MCP servers > Restart extension. bearer_token_env_var reads a token from the environment. Recommended next step is inspect github_standalone in that extension menu and restart VS Code with the token available to its process, then retry a read-only MCP call. GitHub standalone connectivity remains unverified. The two actual Exa calls are recorded in tool-results.json.

Endpoint verification (2026-11-09): Successfully retrieved GitHub official Codex installation guide through exa_standalone / mcp__exa_standalone__web_fetch_exa: https://github.com/github/github-mcp-server/blob/main/docs/installation-guides/install-codex.md. It confirms https://api.githubcopilot.com/mcp/ is the correct hosted endpoint. This validates the URL, not GitHub runtime connectivity. Missing context remains a successful github_standalone call; next verification is tool discovery and a read-only call after server startup.

## Claude Code MCP verification (2026-11-09)

Goal: Verify the `.mcp.json` servers (`github_standalone`, `exa_standalone`) after migrating from Codex to Claude Code.

Scope: This Claude Code session started before `.mcp.json` existed, so neither server's tools were loaded (tool search found no `github_standalone`/`exa_standalone` tools). The only `claude` binary found locally is a Linux build for the app VM and cannot run on macOS, so `claude mcp list` was not possible. Instead, the endpoints were probed directly over MCP Streamable HTTP using the same URLs and headers as `.mcp.json`.

Findings:
- `exa_standalone`: working. `initialize` returned exa-search-server 3.5.0; `tools/list` returned `web_search_exa` and `web_fetch_exa`; a `web_search_exa` call returned results including https://code.claude.com/docs/en/mcp.
- `github_standalone`: endpoint reachable but unauthenticated. HTTP 401 "missing required Authorization header", because `GITHUB_PERSONAL_ACCESS_TOKEN` is not set in this environment.

Hypotheses: With a valid token exported before launching Claude Code, `${GITHUB_PERSONAL_ACCESS_TOKEN}` expansion in `.mcp.json` should satisfy the 401. Not yet observed.

Missing context: An authenticated GitHub MCP call; server loading inside an actual Claude Code session (`/mcp`).

Recommended verification: Export a freshly issued token (the previous one was exposed in plaintext and should be revoked), start Claude Code in this directory, approve the project servers, check `/mcp`, then run a read-only `github_standalone` issue search. The exercise is not verified until both servers return useful results inside Claude Code.
