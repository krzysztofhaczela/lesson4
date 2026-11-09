# Deployment investigation test

Date: 9 November 2026. Scope: a public demonstration, not an investigation of the user's deployment. No repository or incident was supplied, so the agent selected a public GitHub Pages example. No external state was changed.

## Goal

Investigate GitHub Pages reporting “Deployment failed, try again later” during deployment-status polling. Check existing tracking in actions/deploy-pages, retrieve relevant technical information, and recommend the next action.

## Instruction and tool selection

The agent read `AGENTS.md` and `skills/devops-investigate/SKILL.md` after creating them. The rules required external evidence and searching existing tracking. The skill required issue search followed by authoritative information from a different MCP server. Available tool descriptions led to selecting GitHub issue search and Search Service page retrieval. The natural-language goal did not prescribe tool calls.

| Step | Selected MCP tool | Inputs | Observed result and use |
| --- | --- | --- | --- |
| 1 | `mcp__codex_apps__github_search_issues` | `repo:actions/deploy-pages is:issue "Deployment failed, try again later"`, top 3 | Returned issues 418, 406 and 317; established existing reports of this symptom. |
| 2 | `mcp__codex_apps__search_service_web_run` | Open GitHub Docs custom-workflow page and deploy-pages repository README | Retrieved deployment requirements; supplied a checklist against which an actual workflow could be compared. |

Both calls returned `isError: false`. Exact arguments and condensed results are in `tool-results.json`.

## Evidence and correlation

[Issue 418](https://github.com/actions/deploy-pages/issues/418) describes the same error after an initial successful deployment. [Issue 406](https://github.com/actions/deploy-pages/issues/406) includes logs showing artifact discovery and deployment creation before polling failures. These are relevant reports, not proof that a new incident has the same cause. The connector returned null state fields, so open/closed status is unknown.

[GitHub Docs](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages) requires deployment permissions `pages: write` and `id-token: write`, an environment, and a dependency on the build job when jobs are separate. These requirements provide configuration checks; the retrieved issue reports do not prove a permission defect.

The [action README](https://github.com/actions/deploy-pages) confirms prior artifact upload and describes deployment polling and timeout inputs. Inference: because issue 406 got beyond artifact discovery and deployment creation, an absent artifact is a weaker explanation for that specific report. Service-side processing trouble remains a hypothesis. Issue 317 concerns GitHub Enterprise Server and should not be treated as a matching environment without confirming the platform.

## Recommended next action

Inspect the actual failing run's full logs and workflow, comparing the first failure with the last successful run. Verify artifact upload, build dependency, permissions and environment against the documentation. Capture whether failure occurs before deployment creation or during status polling.

If configuration is correct and the run reaches deployment creation, check Pages service health for the failure timestamp and compare the polling signature with existing reports. Use the matching existing issue rather than immediately creating a duplicate; confirm its current state first. After an authorized correction or retry, verify both successful deployment status and the published site. This investigation did not retry or deploy anything.

## Verification and limits

- Two different app-managed MCP integrations returned useful information successfully.
- AGENTS.md contains nine MCP rules; the skill covers search, second-server retrieval, correlation, recommendation and evidence reporting.
- The bundled skill validator was attempted but failed before validation: `ModuleNotFoundError: No module named 'yaml'`. A dependency-free structural check was used instead; it is not equivalent to the bundled validator.
- An initial exploratory artifact/expired search returned loosely related matches. The recorded test narrowed the query to the actual polling error; no artifact-expiration conclusion was drawn.
- No actual user workflow, deployment logs, current service-health evidence or confirmed fix was available. Root cause remains unconfirmed.
- Existing app-managed connections were reused. New standalone endpoint configuration and a fresh-session automatic-routing test were not performed.
