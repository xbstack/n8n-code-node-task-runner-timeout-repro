# n8n Code Node `Task request timed out` / `not matched to a runner` reproduction

Minimal external-task-runner reproduction for the n8n Code node error:

```text
Task request timed out
Your Code node task was not matched to a runner within the timeout period
```

## What this lab verifies

Using official `n8nio/n8n:2.37.1` and `n8nio/runners:2.37.1` images on a Linux x86_64 NAS:

| Case | Result |
| --- | --- |
| External mode, no matching runner | FAIL after the configured 15-second request wait |
| Matching JavaScript runner available | PASS |
| Code request starts first; matching runner registers before deadline | PASS without manually retrying the workflow |

The lab shortens `N8N_RUNNERS_TASK_REQUEST_TIMEOUT` to 15 seconds so the negative control finishes quickly. n8n's current task-runner environment-variable documentation lists a default of 60 seconds.

## Minimal workflow

```text
Manual Trigger
  -> Code

return [{ json: { ok: true, source: 'xbstack-task-runner-repro' } }];
```

The JavaScript itself does no I/O and has no loop. The negative control fails before a runner accepts the task, which separates the error from a long-running JavaScript execution.

## Evidence

- `logs/no-runner.txt` — exact no-runner timeout path.
- `logs/healthy-runner.txt` — matching runner control.
- `logs/runner-started-while-waiting.txt` — pending request succeeds when a runner registers before the request deadline.
- `version-matrix.md` — compact result and evidence boundary.
- `workflow.json` — minimal imported workflow.
- `compose.healthy.yml` — disposable external-runner control.

## Important distinction: request timeout vs task timeout

`N8N_RUNNERS_TASK_REQUEST_TIMEOUT` is the wait-for-runner window: how long a task request can wait for a runner to become available.

`N8N_RUNNERS_TASK_TIMEOUT` is different: it limits how long a task may run after a runner has accepted it.

Increasing the request timeout can help if runners are merely slow or temporarily saturated. It cannot repair a runner that never registers because the sidecar is down, the auth token does not match, the broker is unreachable, the matching runner type fails to start, or the platform control plane has a separate problem.

## Upstream context

Recent reports on 2026-08-25/26 include:

- n8n #37069 — Cloud Code nodes fail with runner timeout.
- n8n #37062 — similar n8n Cloud 2.37.1 report with additional users confirming the symptom.
- n8n #37043 — JavaScript node timeouts in Cloud.
- n8n #37064 — proposed 2.37.x regression hypothesis; maintainer feedback did **not** confirm a universal version regression.
- n8n #36989 — the same high-level message was reported on Cloud 2.35.4.
- n8n #37065 — a narrower external-runner case where scheduled JavaScript is not dispatched while webhook/manual executions succeed. Treat this as a distinct trigger-path problem, not as proof that all runner timeouts share one root cause.

## Official configuration references

- Task runners: https://docs.n8n.io/deploy/host-n8n/configure-n8n/set-up-task-runners/
- Task runner environment variables: https://docs.n8n.io/hosting/configuration/environment-variables/task-runners/
- Hardening task runners: https://docs.n8n.io/deploy/host-n8n/configure-n8n/security/harden-task-runners/

## XBSTACK article

Full bilingual troubleshooting article (published URL after release):

https://www.xbstack.com/en/ai/n8n-code-node-task-runner-timeout/?utm_source=github&utm_medium=referral&utm_campaign=n8n_code_node_task_runner_timeout&utm_content=repository_readme

## Evidence boundary

This repository proves the task-request timeout mechanism in an isolated Docker environment. It does **not** reproduce n8n Cloud's internal orchestration and does not claim that every Cloud timeout is caused by a stopped sidecar. Diagnose runner registration, broker reachability, auth, capacity, startup failure, and platform-specific conditions separately.
