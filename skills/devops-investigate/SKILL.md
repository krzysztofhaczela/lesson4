---
name: devops-investigate
description: Investigate deployment failures by correlating existing issue tracking with authoritative technical documentation through multiple MCP servers.
---

1. Identify the repository, platform, symptoms, and relevant error. Use project context; ask only for missing details that prevent a useful search. Label any public demonstration scenario explicitly.
2. Discover available MCP tools. For this project's standalone assessment, require `github_standalone` and `exa_standalone`; do not substitute built-in app integrations. If unavailable, follow `docs/standalone-setup.md` and report the incomplete setup. Search the issue tracker for the error and related terms; inspect relevant matches and record their returned URLs and status. Do not equate a similar issue with a confirmed root cause or an empty search with proof of no issue.
3. Through a different MCP server, retrieve authoritative documentation relevant to the observed failure. Choose tools by capability, not a fixed invocation embedded in a prompt. If a required server is unavailable, report the gap rather than claiming completion.
4. Correlate symptoms, tracking evidence, and documented requirements. Distinguish confirmed facts, hypotheses, and missing runtime evidence.
5. Recommend the smallest useful next action, with a verification step. Keep investigation read-only unless the user authorizes changes.
6. Report the goal, server/tool names selected, search scope, concise returned evidence with links, how each result influenced the recommendation, and any failures or limitations.
