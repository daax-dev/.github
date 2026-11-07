# What to fix next

Survey of the public [daax-dev](https://github.com/daax-dev) organization on **7 November 2026 (AEDT)**. Nine public repositories, 21 open issues, 27 open pull requests, and no recorded human review on any of them.

The org profile in this repo was last edited on **21 March 2026, 11:57pm AEDT**. It names one project. The site at [daax.dev](https://daax.dev) still leads with that same project. The newest blog post is from February 2026.

## Snapshot

| Repo | Last push (AEDT) | Open issues | Open PRs | Main health | License |
|---|---|---|---|---|---|
| [daax-web](https://github.com/daax-dev/daax-web) | 4 Oct 2026, 11:36am | 4 | 6 | CI runs on pull requests only. Last image publish on main succeeded 27 Sep 2026. | none |
| [nanofuse](https://github.com/daax-dev/nanofuse) | 6 Oct 2026, 8:06pm | 3 | 13 (all Dependabot) | Last green `CI/CD Pipeline` on main: 12 Jul 2026. Every open dep PR is unstable. | MIT |
| [daax-devtools](https://github.com/daax-dev/daax-devtools) | 6 Sep 2026, 12:48pm | 5 | 3 | Image publish on main has been red since 12 Jul 2026. | none |
| [agentic-retrospective](https://github.com/daax-dev/agentic-retrospective) | 4 Sep 2026, 10:25am | 8 | 3 | npm `0.1.4` published 9 May 2026. | none |
| [agent-handoff](https://github.com/daax-dev/agent-handoff) | 12 Jul 2026, 6:06am | 0 | 0 | CI green. npm `0.1.6` published 9 May 2026, behind main. | none |
| [vagrant-skill](https://github.com/daax-dev/vagrant-skill) | 11 Jul 2026, 1:10pm | 0 | 1 | CI green. npm `0.7.4` published 25 Mar 2026. | Apache-2.0 |
| [learn](https://github.com/daax-dev/learn) | 10 Jul 2026, 11:50pm | 0 | 0 | No CI workflow. | none |
| [gitloop](https://github.com/daax-dev/gitloop) | 29 May 2026, 1:38pm | 1 | 1 | No CI workflow. | none |
| [.github](https://github.com/daax-dev/.github) (this repo) | 21 Mar 2026, 11:57pm | 0 | 0 | Profile only. Org description and website URL are empty. | none |

Pinned on the org page today: `agentic-retrospective` and `vagrant-skill`.

No repository has a `SECURITY.md`. Six of nine have no license.

## Do these first

Work in this order. Each item is already open; none of it is a new feature.

### 1. Land the daax-web security fix

[daax-web#482](https://github.com/daax-dev/daax-web/pull/482) — `security: remediate premortem findings A2–A10 + R1–R6`.

Opened 2 Aug 2026. Checks were green (lint, typecheck, test, build, auth audit, image build, Trivy). It now **conflicts** with main. Copilot commented; nobody approved it.

The diff covers the findings that are code fixes:

- SSRF guard on MCP URL registration (loopback, link-local, metadata, private ranges).
- `requireAuth` on devcontainer, backlog, and workflow-editor handlers.
- MCP child processes stop inheriting the full `process.env`.
- `docker exec` limited to daax session names; `docker` / `docker-compose` removed from the inspector launcher allowlist.
- Deploy compose fails closed unless `DAAX_REQUIRE_AUTH` is set.
- Path confinement on backlog status and workspace base paths.

A1 (flat authorization: any Pocket ID identity is a full operator) is written down as an intentional single-operator posture. Leave that decision alone.

**Next action:** rebase onto current main, re-run CI, review the conflict resolution, merge.

### 2. Make logout end the session

[daax-web#510](https://github.com/daax-dev/daax-web/pull/510) — logout POSTs to forward-auth, then this host's Pocket ID.

Also **conflicting**. Same auth surface as #482. Rebase it after #482 lands so the two patches do not fight.

### 3. Get a fresh error out of daax-devtools publish

`Publish devcontainer images` failed on main on 12 Jul 2026 in the `code-server` job (`./.github/actions/build-scan-push`). The log is gone (GitHub returns HTTP 410).

**Next action:** re-run the workflow on main, keep the log, fix the code-server build. The two large feature PRs below stay blocked until publish is green.

### 4. One nanofuse security bump, then the bot queue

Main's pipeline last passed on 12 Jul 2026. Since then only Dependabot has pushed. Thirteen dep PRs sit open, oldest 114 days, every one `UNSTABLE`.

The 6 Oct 2026 run on the Ubuntu 25.10 bump failed in **Security Scanning**:

- Reachable `govulncheck` finding [GO-2026-5970](https://pkg.go.dev/vuln/GO-2026-5970) via `internal/layer/scaffold.go` (`golang.org/x/text`).
- Trivy: `google.golang.org/grpc` CVE-2026-33186 (CRITICAL, fix 1.79.3) and CVE-2026-84304 (HIGH, fix 1.83.1); `golang.org/x/crypto` CVE-2024-45337; `golang.org/x/net` CVE-2026-46600.

The `x/text`, `x/sys`, and `x/sync` bumps also fail **Build and Test**, **Lint**, and **Quick Validation** in under 30 seconds. That is fast enough to be a broken workflow or toolchain pin, separate from the CVE itself.

**Next action:**

1. Find out why those three jobs die in seconds. Fix that before merging any bot PR.
2. Open one human PR that bumps `golang.org/x/text`, `google.golang.org/grpc`, `golang.org/x/net`, and `golang.org/x/crypto` to the fixed versions, and confirm `govulncheck` and Trivy go green.
3. Close or rebase the thirteen Dependabot PRs after that. Leave the Ubuntu 25.10 base-image bump until the image bugs in section 6 are fixed. A newer Ubuntu tag on a rootfs that does not build is the wrong order.

### 5. Decide the 26,000-line workers pull request

[daax-web#523](https://github.com/daax-dev/daax-web/pull/523) — digital workers module with a Technical Project Manager template.

Opened 4 Oct 2026. **Mergeable, CI green**, +26,011 / −20 across 77 files. It has had no review.

Merging it as-is buries the security rebase and makes #482 and #510 harder. Leaving it open another month guarantees a second conflict cycle.

**Next action:** a human reads it this week and either splits it into reviewable slices or schedules a dedicated review after #482 and #510 merge. It does not jump the queue.

## Product bugs already filed

These are the defects behind the story the homepage tells: shared context, recorded sessions, sandboxed execution.

### 6. Agent View on Linux hosts

[daax-web#494](https://github.com/daax-dev/daax-web/issues/494), open since 9 Sep 2026, no follow-up.

Agent View works on macOS because Docker Desktop presents the container's connection on agentd's loopback listener (`127.0.0.1:7717`), and local trust is based on that listener. On Linux the container reaches the host over the bridge, so either nothing is listening or the listener is off-loopback and every request needs a session. `fetchDaemon` sends no credential, so those reads are 401.

This is the "watch the agent" surface, broken on the fleet hosts.

### 7. Nanofuse images do not produce a bootable OS

Three issues, all filed 11–12 Jul 2026, no comments since:

- [#309](https://github.com/daax-dev/nanofuse/issues/309) — `local://` Dockerfile layers are tarred as source and never built, so the composed rootfs has no real OS.
- [#310](https://github.com/daax-dev/nanofuse/issues/310) — image-build CLI fails on a CGO-less build because of the SQLite layer-cache stub, including with `--no-cache`.
- [#274](https://github.com/daax-dev/nanofuse/issues/274) — `serial-getty@ttyS0` waits on `dev-ttyS0.device` and adds about 90 seconds to every boot.

Fix #309 and #310 before any base-image upgrade. #274 is a boot-time tax on every sandbox start; it belongs in the same pass.

### 8. Herdr launcher and the two stale agent-image fixes

- [daax-web#473](https://github.com/daax-dev/daax-web/issues/473) — add a Herdr + Claude AI Coding launcher. Open since 7 Jul 2026.
- [daax-web#485](https://github.com/daax-dev/daax-web/pull/485) and [#486](https://github.com/daax-dev/daax-web/pull/486) — run Herdr on an image that contains it, and resolve the Copilot entrypoint at runtime. Both **conflict**. They have been open since mid-August.

Rebase them into one small PR after the security work, or close them and re-cut against current main. Two conflicting PRs for the same launcher is how the issue stays open for another quarter.

### 9. gitloop sandbox fix, plus a test workflow

[gitloop#4](https://github.com/daax-dev/gitloop/pull/4) is mergeable: honor runtime pipeline config and repair stale `PORT` references. Open since 29 May 2026. The repo has no CI, so "mergeable" means GitHub sees no conflicts, not that anything ran.

[gitloop#1](https://github.com/daax-dev/gitloop/issues/1) ("GitLoop Local") has had no update since 27 May 2026.

**Next action:** add a minimal CI workflow (typecheck and unit tests on pull request), then land or close #4. If #1 is still the product direction, rewrite it as a checklist; otherwise close it.

## Finish work that is already written

### 10. Retrospective telemetry, then publish

Epic [#44](https://github.com/daax-dev/agentic-retrospective/issues/44) (Claude-native telemetry) has seven child issues open since late May, all untouched. Two of them already have mergeable PRs:

- [#56](https://github.com/daax-dev/agentic-retrospective/pull/56) — token and cost accounting ([#37](https://github.com/daax-dev/agentic-retrospective/issues/37)).
- [#57](https://github.com/daax-dev/agentic-retrospective/pull/57) — waste and efficiency detectors ([#39](https://github.com/daax-dev/agentic-retrospective/issues/39)).

npm is still `0.1.4` from 9 May 2026. Review those two PRs, merge what holds up, and publish. Leave #40, #41, #42, and #43 (delivery correlation, ghost assets, durable ledger, transcript export) until those two are shipped.

[#19](https://github.com/daax-dev/agentic-retrospective/issues/19) (multi-repo support) is still open even though PR #29, titled for that issue, merged on 9 May 2026. Read the issue against main and either close it or list the remaining gap in one comment.

### 11. daax-devtools feature PRs, after publish is green

Both are mergeable and have been waiting since about 18 Apr 2026:

- [#10](https://github.com/daax-dev/daax-devtools/pull/10) — docker-agent lifecycle, copy-on-write snapshots, Claude Code hot reload (+1,124).
- [#11](https://github.com/daax-dev/daax-devtools/pull/11) — testcontainers (microVM isolation, evals, reporting) and ai-coding-containers (+2,172).

[#3](https://github.com/daax-dev/daax-devtools/pull/3) conflicts and is older. Close it if #10 and #11 cover it.

The five open devtools issues (#4, #5, #6, #8, #9: docker-agent compatibility, Hawkeye policy injection, readiness probes, Hawkeye lifecycle events, container pooling) were filed in April and never moved. They describe the same area as #10 and #11. Review the PRs against those issues and close the issues the PRs already solve.

### 12. Small docs and config that can land in an afternoon

- [daax-web#508](https://github.com/daax-dev/daax-web/pull/508) — how to add a homepage feature card. Mergeable, CI green, docs only.
- [agentic-retrospective#59](https://github.com/daax-dev/agentic-retrospective/pull/59) — closed-loop delivery playbook. The same doc merged in `agent-handoff` and `vagrant-skill` on 11 Jul 2026. This copy is still open. Land it or drop it so the three repos match.
- [vagrant-skill#21](https://github.com/daax-dev/vagrant-skill/pull/21) — Claude Code `config.yml` trigger policy. Mergeable since 18 Apr 2026.

## The public face of the org

This repo is the org profile. It is the first thing a visitor reads, and it is wrong.

### 13. Rewrite `profile/README.md`

The Projects table lists only [agentic-retrospective](https://github.com/daax-dev/agentic-retrospective). Add the rest, with an honest status so the page does not oversell:

| Repository | What it is | Status to show |
|---|---|---|
| daax-web | Browser workbench: terminal, AI coding tools, session record and replay | Active. Security patch and Linux Agent View still open. |
| nanofuse | Firecracker microVM sandbox | Active. Image build and boot bugs open. Deps behind on known CVEs. |
| daax-devtools | Devcontainers, testcontainers, image publish | Publish pipeline red since July 2026. |
| agent-handoff | MCP handoff to a background agent | Shipped. CI green. npm lags main. |
| agentic-retrospective | Evidence-based retrospectives | Shipped to npm 0.1.4. Telemetry PRs waiting. |
| gitloop | Git-tag inner loop for agent handoff | Stalled since May 2026. No CI. |
| vagrant-skill | Disposable Ubuntu VMs as an agent skill | Shipped. One config PR open since April. |
| learn | Notes and exercises for AI coding practice | No CI, no license, last touch July 2026. |

Keep the problem / approach copy. It still matches the site. The project list is the part that drifted.

### 14. Fill in the org record and the pins

On the GitHub org (needs an owner in the UI; this repo cannot set it):

- Website: `https://daax.dev`
- Description: the one-line already used in this repo's description ("secure agentic pair programming where developers and AI agents work together").
- Pin `daax-web` and `nanofuse` next to `agentic-retrospective`. `vagrant-skill` can stay as a fourth pin or give up its slot.
- Add topics on each repo (`mcp`, `sandbox`, `firecracker`, `devcontainer`, `agents`) so search finds them.

### 15. Security policy and licenses

Add, in this `.github` repo so they apply org-wide:

- `SECURITY.md` with a private contact for reports. Right now every repo invites a public issue for a vulnerability.
- A license decision. `nanofuse` is MIT, `vagrant-skill` is Apache-2.0, and daax-web, daax-devtools, agent-handoff, agentic-retrospective, learn, gitloop, and this repo have none. Pick one default and add the file to the repos that lack it.
- A short `CONTRIBUTING.md` and one issue template. Only retrospective and agent-handoff have issue templates today.

### 16. Publishing and CI gaps

- `agent-handoff` main moved through 11 Jul 2026; npm stayed at 0.1.6 from 9 May. Either cut a release or stop the README from implying the npm package matches main.
- Retrospective publish follows the telemetry merges in item 10.
- Add CI to `learn` and `gitloop`.
- `daax-web` CI triggers on `pull_request` and `workflow_dispatch`, so a bad merge to main raises no CI run. Add a `push` to `main`, or document the branch-protection required check if that was deliberate.
- `daax-devtools` has publish workflows and no workflow named CI. A pull request can be "green" while the thing users pull (the image) is the job that fails.

## Approved epics to schedule, after the list above

These are marked approved and have not moved. They are real work. They are not next.

| Item | Age | What it is |
|---|---|---|
| [daax-web#105](https://github.com/daax-dev/daax-web/issues/105) | 146 days | brain2daax platform-maturity rollout |
| [daax-web#157](https://github.com/daax-dev/daax-web/issues/157) | 135 days | Agent Attention Layer |
| [retrospective#44](https://github.com/daax-dev/agentic-retrospective/issues/44) | 166 days | Claude-native telemetry epic (children #40–#43 still have no PR) |

When items 1–8 are done, pick **one** of these and write the first slice as issues with owners. The other two stay approved and untouched on purpose, so the board stops looking like three programs in parallel.

## Sequence

```text
1. Rebase and merge daax-web #482 (security), then #510 (logout).
2. Re-run daax-devtools image publish. Fix the code-server failure.
3. Repair nanofuse CI, then one dependency PR for x/text, grpc, x/net, x/crypto.
4. daax-web #494 (Agent View on Linux) and nanofuse #309, #310, #274 (images and boot).
5. Review retrospective #56 and #57. Publish npm if they land.
6. Rewrite profile/README.md. Set the org URL. Add SECURITY.md. Choose a license.
7. Then the large feature PRs: daax-web #523, daax-devtools #10 and #11.
```

## Defer

- New product surface (attention layer, brain2daax, the digital-workers module) while #482 conflicts and Agent View fails on Linux.
- Merging thirteen Dependabot PRs one at a time, and moving the base image to Ubuntu 25.10, before the rootfs bugs and the vuln gate are fixed.
- A new blog post. The last one is February 2026. The profile and the security patch should match reality before the site announces more of it.
