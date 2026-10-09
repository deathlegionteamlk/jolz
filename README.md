<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:000000,100:00D9FF&height=200&section=header&text=deathforge&fontSize=70&fontColor=FFFFFF&animation=fadeIn&fontAlignY=38&desc=Concurrent%20AI%20Agent%20Orchestration&descAlignY=58&descSize=18" width="100%"/>

<img src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&size=20&duration=3000&pause=800&color=00D9FF&center=true&vCenter=true&width=600&lines=No+async+libraries.+No+job+queues.;Just+Fibers%2C+Channels%2C+and+cgroups.;Built+from+the+stdlib+up." alt="typing" />

[![Language](https://img.shields.io/badge/Crystal-1.14%2B-black.svg?style=for-the-badge)](https://crystal-lang.org)
[![Platform](https://img.shields.io/badge/Platform-Linux-black.svg?style=for-the-badge)](#requirements)
[![License: MIT](https://img.shields.io/badge/License-MIT-000000.svg?style=for-the-badge)](LICENSE)

</div>

---

## What it is

A concurrent AI agent orchestration runtime, written in Crystal, with nothing borrowed. No async library, no job-queue shard, no web framework. Fibers and Channels handle concurrency. Raw `lib C` FFI talks to cgroups v2 for sandboxing. The SSE layer runs on Crystal's built-in `HTTP::Server`.

On top of that: priority dispatch, task pipelines, retry policies, YAML-defined workflows, cancellation, rate limits, structured logging, and graceful shutdown.

## Architecture

### Scheduler

The scheduler is the dispatch loop. It holds a registry of `Executor` instances and a priority queue that accepts `TaskRequest` structs, higher `priority` goes first. A long-running Fiber pulls from the queue and spawns worker Fibers up to a configurable concurrency limit.

Each worker calls the executor's `call` method, captures the `ExecutionResult`, and writes it back through a per-task Channel. Nothing gets dropped: if concurrency is maxed out, the dispatch loop sleeps and retries until a slot frees up.

Two dispatch modes:

- `submit`: fire the task and return immediately. The caller picks up results through SSE broadcast, a shared channel, or a collector.
- `submit_sync`: block the calling Fiber until the executor returns. Useful when an agent needs the output before deciding its next move.

Every completed result lands in a ring-buffer `ResultStore`, which backs the `/history` and `/metrics` endpoints.

### Sandbox

Process isolation goes through Crystal's `Process` module plus direct `lib C` bindings to cgroups v2. The sandbox forks a child for each tool execution, drops the PID into a dedicated cgroup subtree under `/sys/fs/cgroup/deathforge`, and sets memory and CPU limits before the executor runs.

`CgroupSandbox` creates one directory per task, writes `memory.max`, `memory.swap.max` (0, no swap), and `cpu.max` (quota/period or `"max"`). It delegates `memory` and `cpu` controllers to child subtrees through `cgroup.subtree_control`.

The parent polls with non-blocking `waitpid` (`WNOHANG`) against a monotonic deadline. Deadline passes, it sends `SIGKILL` and raises `SandboxTimeout`. It also watches `memory.events` for `oom_kill` counters. If the kernel's OOM killer stepped in, it raises `SandboxOOM`.

Whatever the outcome, `cleanup` kills any leftover procs and deletes the cgroup directory. No orphaned subtrees.

### SSE layer

The SSE server runs on Crystal's built-in `HTTP::Server`. Nine routes, no framework:

- `/stream`: long-lived SSE connection. Pushes `result` and `pipeline_result` events as tasks finish, with keepalive comments so proxies don't time it out.
- `/submit`: JSON with `tool`, optional `id` and `priority`, plus arbitrary args. Dispatches and returns `{ok: true, id: ...}` right away.
- `/cancel`: JSON with `id`. Flags the task; the scheduler checks the flag before and during execution.
- `/pipeline`: JSON with a `steps` array (`tool`, `args`, optional `label`). Runs the chain, streams the aggregate result.
- `/workflow`: YAML with a `stages` list. Supports `condition` (success/failure) and `parallel_with` for concurrent branches. `{{label.output}}` injects upstream results downstream.
- `/status`: active/pending counts, executor list, cancelled task count.
- `/health`: `{status: "alive", version: ...}`.
- `/metrics`: result counts, success/failure ratio, average latency, active/pending counts, connected SSE client count.
- `/history`: recent results, `?limit=N&tool=name` supported.

Client connections live in a Mutex-protected array. Broadcast walks the array, writes the SSE wire format, and drops closed connections as it goes.

### Pipeline

A `Pipeline` chains tool calls where each step consumes the previous one's output through `args["input"]`. It stops on first failure or runs to completion. `PipelineResult` carries every step's result in order, a `success` flag, and total elapsed time.

```crystal
pipeline = Deathforge::Pipeline.new(scheduler)
pipeline.add_step("shell", {"command" => "echo hello"})
pipeline.add_step("input_echo")
result = pipeline.run
```

Step two gets `"hello\n"` as `args["input"]`.

### Workflow

A `Workflow` is a YAML-defined multi-step plan with conditional branching and parallel stages. `{{label.output}}` resolves to prior stage results. `condition` (`success`, `failure`) gates whether a stage runs. `parallel_with` groups stages meant to run concurrently.

```yaml
stages:
  - tool: shell
    args: {command: "echo data"}
    label: fetch
  - tool: file_write
    args: {path: "/tmp/out", content: "{{fetch.output}}"}
    condition: success
    label: save
```

### Retry

`RetryPolicy` sets max retries, backoff (fixed, exponential, linear), and which exit codes are worth retrying. `Retrier` wraps `submit_sync` and applies the policy on failure. Presets: `no_retry`, `aggressive`, `conservative`.

```crystal
retrier = Deathforge::Retrier.new(scheduler, Deathforge::RetryPolicy.aggressive)
result = retrier.execute(request)
```

### Rate limiter

Per-executor sliding window limits. `set_limit("tool_a", 5, 10)` allows 5 calls per 10 seconds. `wait_and_allow` blocks until a slot opens. Executors with no configured limit just proceed.

### Cancellation

`CancelRegistry` tracks cancelled task IDs. The scheduler checks it before dispatch and after execution. Cancelled tasks come back with exit code -8. `/cancel` marks a task and broadcasts an SSE event.

### Logging

Structured JSON line logging to STDERR, or any IO you point it at. Each entry has timestamp, level, message, metadata. Levels: DEBUG, INFO, WARN, ERROR, FATAL. Swap the output with `Logging.set_output(io, level)`.

### Shutdown

`ShutdownHandler` traps SIGINT and SIGTERM. First signal stops the SSE server, drains the scheduler, then exits. Second signal force-exits immediately.

## Executor interface

`Deathforge::Executor` is an abstract class. Subclass it to add a tool. One required method:

```crystal
def call(args : Hash(String, String)) : ExecutionResult
```

Optional overrides:

- `timeout_ms`: wall-clock limit before the sandbox kills the process. Default 30 seconds.
- `memory_limit_bytes`: cgroup `memory.max`. Default 512 MiB.
- `cpu_quota_us`: cgroup `cpu.max` quota in microseconds. Default nil (unlimited).

Built-in executors:

| Name | Key args | Description |
|------|----------|-------------|
| `shell` | `command` | Run a shell command, capture stdout/stderr |
| `file_read` | `path` | Read a file from disk |
| `file_write` | `path`, `content` | Write content to a file |
| `http_fetch` | `url` | HTTP GET, return response body |

Register a custom executor:

```crystal
scheduler = Deathforge::Scheduler.new
scheduler.register(MyTool.new)
scheduler.run
```

Set a rate limit on an executor:

```crystal
scheduler.rate_limiter.set_limit("shell", max_calls: 10, window_seconds: 60)
```

## Usage

Start the runtime:

```crystal
require "deathforge"

server = Deathforge.run(port: 8080, concurrency: 8)
```

This registers the built-in executors, starts the scheduler, binds the SSE server, and installs signal handlers. The process runs until SIGINT or SIGTERM.

```bash
# submit a task
curl -X POST http://localhost:8080/submit \
  -d '{"tool":"shell","command":"echo hello","id":"t1","priority":"5"}'

# cancel it
curl -X POST http://localhost:8080/cancel -d '{"id":"t1"}'

# run a pipeline
curl -X POST http://localhost:8080/pipeline \
  -d '{"steps":[{"tool":"shell","args":{"command":"echo data"},"label":"gen"},{"tool":"input_echo"}]}'

# run a workflow
curl -X POST http://localhost:8080/workflow \
  -d 'stages:
  - tool: shell
    args: {command: "echo hello"}
    label: greet
  - tool: input_echo
    condition: success'

# stream results, check metrics, view history
curl -N http://localhost:8080/stream
curl http://localhost:8080/metrics
curl "http://localhost:8080/history?limit=20&tool=shell"
```

## Building

```bash
make deps    # install shards (none required right now)
make build   # compile release binary
make test    # run specs
make run     # build + run
make format  # auto-format source
make clean   # remove binary and lib/
```

## Project structure

```
deathforge/
  shard.yml             shard manifest
  Makefile              build + run tasks
  src/
    deathforge.cr       entry point, top-level API
    deathforge/
      executor.cr       abstract Executor class, ExecutionResult
      scheduler.cr      priority-aware Fiber scheduler with Channel dispatch
      cgroup.cr         lib C FFI + CgroupSandbox helper
      sandbox.cr        fork + cgroup + timeout + OOM detection
      sse_server.cr     SSE HTTP endpoint, nine routes
      builtin.cr        shell, file_read, file_write, http_fetch
      logger.cr         structured JSON line logging
      result_store.cr   ring buffer for execution history
      retry.cr          RetryPolicy, Retrier, backoff strategies
      pipeline.cr       task chaining with output injection
      workflow.cr       YAML-based multi-step workflow DSL
      shutdown.cr       SIGINT/SIGTERM handler, graceful drain
      rate_limiter.cr   sliding window per-executor rate limits
      cancel.cr         CancelRegistry for task cancellation
  spec/
    spec_helper.cr      stub executors for test isolation
    scheduler_spec.cr   scheduler dispatch, concurrency, priority
    sandbox_spec.cr     cgroup setup, cleanup, and error specs
    result_store_spec.cr ring buffer storage and retrieval
    retry_spec.cr       backoff strategies, retry policies
    pipeline_spec.cr    chaining, output injection, failure stop
    features_spec.cr    cancellation, rate limits, logging, shutdown
```

## Requirements

- Crystal >= 1.14.0
- Linux with cgroups v2 mounted at `/sys/fs/cgroup`
- Root or cgroup write access for sandbox mode (non-sandboxed mode works without privileges)

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:00D9FF,100:000000&height=120&section=footer&animation=fadeIn"/>

**DEATHFORGE // DEATH LEGION TEAM**

</div>
