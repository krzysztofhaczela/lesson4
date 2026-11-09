# Investigation: GitHub Pages "Deployment failed, try again later"

> **Public demonstration scenario.** No specific affected repository, workflow run, or log was supplied. Findings come from public tracker data and vendor documentation only.

Date: 2026-11-09

## Goal

Determine whether the symptom "GitHub Pages deployment fails with `Deployment failed, try again later` while polling deployment status" is already tracked in `actions/deploy-pages`, gather authoritative documentation on deployment requirements, and recommend the next diagnostic action.

## Scope

- Issue tracker: `actions/deploy-pages` (open and closed issues), via `github_standalone`.
- Action source: `actions/deploy-pages` default branch, to see where the error message comes from, via `github_standalone`.
- Documentation: docs.github.com (Pages custom workflows, publishing source, Pages REST API) and the `actions/deploy-pages` README, via `exa_standalone`.
- Read-only. No issues, comments, re-runs, or API writes were made.

## Servers and tools used

| Server | Tool | Purpose |
|---|---|---|
| `github_standalone` | `search_issues` | Search the `actions/deploy-pages` tracker for the error text |
| `github_standalone` | `issue_read` (get, get_comments) | Inspect issues #418 and #406 |
| `github_standalone` | `search_code` | Find where the error string is emitted |
| `exa_standalone` | `web_search_exa` (x2) | Find official Pages deployment requirements and publishing-source docs |
| `exa_standalone` | `web_fetch_exa` (x2) | Read the Pages REST API reference (the second fetch used a larger limit because the first was truncated) |

## Findings

### Confirmed facts

1. **What the error means.** In `actions/deploy-pages` `src/internal/deployment.js`, the `finalErrorStatus` map translates the Pages API status `deployment_failed` into `"Deployment failed, try again later."`. Artifact problems use a different status, `deployment_content_failed` ("Artifact could not be deployed… hard links, symlinks and total size is less than 10GB"). So this message means the Pages backend reported a **generic** failure, not a rejected artifact.
2. **It is already tracked, and the issue is still open.**
   - [#418 "Error: Deployment failed, try again later."](https://github.com/actions/deploy-pages/issues/418): **open**, 19 comments, 10 👍, opened 2026-05-24, last updated 2026-07-06, no linked fix PR. In the logs posted there, the artifact was found and the deployment was created before the error. Many reports cluster around 2026-07-02 to 2026-07-06. Several reporters say it cleared on its own or after retries. It affects both `deploy-pages@v4` and `@v5`, and both "GitHub Actions" and "Deploy from a branch" sources. In one debug log, `GITHUB_TOKEN` permissions show `Pages: write`.
   - [#406 "Intermittent Deploy Failures starting 2026-01-13"](https://github.com/actions/deploy-pages/issues/406): **open**, 2 comments, last updated 2026-11-06. Reports the same final error after polling stuck at `syncing_files`, plus related timeouts stuck at `deployment_queued`. One commenter says a stale earlier deployment was still locked internally, and the job failed even after they cancelled it.
   - Related closed issues (not the same symptom): [#372](https://github.com/actions/deploy-pages/issues/372) (multiple `github-pages` artifacts), [#166](https://github.com/actions/deploy-pages/issues/166) (blank status treated as an error), [#163](https://github.com/actions/deploy-pages/issues/163) (deployment cancelled).
3. **Documented requirements** ([Using custom workflows with GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages), [deploy-pages README](https://github.com/actions/deploy-pages?tab=readme-ov-file)):
   - The deploy job needs at least `pages: write` and `id-token: write`.
   - `needs:` must point to the build job.
   - The job must use an `environment`, by default `github-pages`.
   - The artifact must be a gzip archive containing a single tar file, under 10 GB, with no symbolic or hard links.
   - With a branch source, the deployment must come from that branch unless environment protection rules override this.
4. **Publishing source** ([Configuring a publishing source](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)): the source can be "Deploy from a branch" or "GitHub Actions". Branch-source sites also deploy through a GitHub Actions run (`pages-build-deployment`). Commits pushed with `GITHUB_TOKEN` do not trigger a Pages build.
5. **Pages REST API** ([REST API endpoints for Pages](https://docs.github.com/en/rest/pages/pages)):
   - `GET /repos/{owner}/{repo}/pages/deployments/{pages_deployment_id}` returns `status`, which is one of `deployment_in_progress`, `syncing_files`, `finished_file_sync`, `updating_pages`, `purging_cdn`, `deployment_cancelled`, `deployment_failed`, `deployment_content_failed`, `deployment_attempt_error`, `deployment_lost`, or `succeed`. The ID can be the commit SHA.
   - `GET /repos/{owner}/{repo}/pages` returns `build_type` (`legacy`/`workflow`), `source`, `status`, `cname`, and `https_certificate.state`.
   - `GET /repos/{owner}/{repo}/pages/health` checks DNS for custom domains.
   - `POST …/pages/builds` requests a build. Builds are limited to one concurrent build per repo and per requester.

### Hypotheses (not confirmed)

- **H1, most likely:** an intermittent or backend-side Pages deployment failure, the same thing tracked in #418 and #406. This fits because the artifact and deployment were created first, the error is the generic `deployment_failed` and not `deployment_content_failed`, and the failures clustered and then cleared without changes. Neither issue has a maintainer root cause or fix.
- **H2:** stale or locked Pages deployment state, or a mismatch between configured `build_type` and the workflow actually deploying. Comments in #406 and #418 suggest this, for example a recent source migration or a stale queued deployment. Some users report that re-applying settings or `POST /pages/builds` helped, but that is anecdotal.
- **H3:** concurrent or overlapping deployments. One user reported a fix with `concurrency.cancel-in-progress: true`, but another hit the error with no concurrency at all.
- **Less likely:** missing permissions or an invalid artifact. These normally produce different errors, such as permission or OIDC errors or `deployment_content_failed`. Still check them because they are cheap to rule out.

A similar issue does not prove a shared root cause. Issue comments are user reports, not maintainer diagnoses.

## Missing context

- The affected repository, workflow file, run and job IDs, and the full deploy-step log. Ideally a re-run with `ACTIONS_STEP_DEBUG=true`.
- The configured Pages source and `build_type`, and whether either changed recently.
- Whether a custom domain is set, and its certificate and DNS state.
- The `deploy-pages` version, the concurrency settings, and whether overlapping runs or in-place re-runs happened.
- githubstatus.com incidents for Pages and Actions at the failure time. These were not checked: no status-page tool was used.
- Any maintainer statement on #418 or #406. None was found in the comments retrieved.

## Recommended next diagnostic action

Read-only. Running these against a real repository needs credentials the user controls.

1. **Read the server-side status and the site configuration** for the failed deployment (the SHA appears in the log line `Created deployment for <sha>`):
   ```
   GET /repos/OWNER/REPO/pages/deployments/<sha>
   GET /repos/OWNER/REPO/pages
   ```
   - `deployment_failed` or `deployment_lost` with a matching `build_type`, permissions, and environment points to H1, a backend issue. Check githubstatus.com, then re-run the **whole workflow** or push a new commit, not just the deploy job, to avoid the multiple-artifact error described in #418.
   - A `build_type` that doesn't match the workflow in use, or a stale earlier deployment still in progress, points to H2.
   - `deployment_content_failed` means the artifact itself is the problem. Inspect the tar for symlinks or hard links and check its size.
2. **Statically check the workflow** against the documented requirements: `pages: write` and `id-token: write`, `needs: build`, `environment: github-pages`, a single `github-pages` artifact, and a `concurrency` group.

### Verification step

The fix is verified when a fresh run completes `actions/deploy-pages` with a final `succeed` status, and `GET /repos/OWNER/REPO/pages/deployments/<new-sha>` returns `status: succeed`. If the error recurs with matching configuration and no status incident, add the run ID and API status as evidence on #418. Only do this with the user's explicit authorization, because it is an external write.
