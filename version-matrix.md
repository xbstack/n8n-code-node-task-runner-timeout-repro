# n8n Code Node task-runner timeout matrix

Test date: 2026-08-26

| Case | n8n | Runner image | Request timeout | Result | Evidence |
| --- | --- | --- | ---: | --- | --- |
| External mode, no matching runner | 2.37.1 | none | 15s | FAIL after ~15.04s with `Task request timed out` and `Your Code node task was not matched to a runner...` | `logs/no-runner.txt` |
| External mode, matching JS runner available | 2.37.1 | 2.37.1 | 15s | PASS, Code returns `{ok:true}` | `logs/healthy-runner.txt` |
| External mode, request starts before runner; matching runner starts before deadline | 2.37.1 | 2.37.1 | 15s | PASS after ~8.42s; pending task is accepted after JS runner registers | `logs/runner-started-while-waiting.txt` |

## What this proves

- The observed `Task request timed out` path is a broker/requester timeout while waiting for an available matching task runner.
- The same minimal JavaScript succeeds on the same n8n version once a matching JS runner registers.
- `N8N_RUNNERS_TASK_REQUEST_TIMEOUT` controls how long the requester waits for a runner; increasing it does not repair a runner that never becomes available.

## What this does not prove

- It does not prove that n8n 2.37.1 has a universal Cloud regression.
- It does not prove that every report with the same error is caused by a stopped container; authentication mismatch, broker reachability, runner capacity, startup failure, or platform-specific control-plane problems can produce the same high-level symptom.
- It does not reproduce n8n Cloud internals; the experiment uses isolated Docker containers on a Linux x86_64 NAS.

## Current upstream context

On 2026-08-25/26, multiple n8n issues reported Code-node runner timeouts, including #37069, #37062, #37043, #37064 and #36989. However, maintainer feedback on #37064 explicitly did not confirm a universal 2.37.x regression. Separately, #37065 reports a narrower external-runner dispatch difference where scheduled JavaScript executions fail while equivalent webhook/manual executions succeed; that is a distinct search task and should not be conflated with the generic no-runner timeout reproduced here.
