---
name: ai-speedup
description: >-
  This skill should be used when a user wants to make AI / model / agent tasks
  faster, complains that AI is slow, or asks about concurrency, async,
  streaming, or connection pooling for AI workloads. It provides a decision
  framework that classifies the bottleneck (independent-parallel vs I/O-wait vs
  compute-bound vs rate-limited) and selects the correct acceleration technique,
  then supplies ready-to-use prompts and Python/JS code snippets to apply it.
  Trigger phrases include "AI 太慢", "提速", "让 AI 快一点", "并发", "流式输出",
  "异步", "连接池", "batch", "concurrency", "streaming".
agent_created: true
---

# AI Speedup Strategist

## Purpose

Make AI / model / agent tasks faster by correctly diagnosing the bottleneck and
applying the matching acceleration technique — instead of blindly increasing
concurrency, which only raises throughput, never single-request latency.

## When to use

- The user complains AI is slow, wants it faster, or asks how to speed up AI work.
- The user mentions concurrency / async / streaming / connection pool / batch for model or API calls.
- Before designing a workflow, agent, or batch job that calls models or external APIs.
- When auditing an existing pipeline for speed.

## Core framework

Classify every task along three axes before acting:

1. **Task structure** — multiple independent subtasks, or a single dependent chain / single response.
2. **Bottleneck type** — I/O wait (waiting on API / file / network) vs compute (model token-by-token generation, local heavy compute) vs external rate limit (quota / 429).
3. **Dependency** — does step N+1 require step N's result?

### Selection rules (follow strictly)

- **Multiple independent subtasks** → concurrency / batch / multi-worker. Raises throughput and lowers total wall-clock for the batch.
- **Single task + I/O wait** → async non-blocking + streaming output + connection pool.
- **Single task + compute bottleneck** → streaming (perceived speed only) + check whether the task splits into independent subtasks. If it splits, fall back to concurrency; if not (a single dependent chain), keep streaming and do NOT raise concurrency.
- **Dependent chain** → must be serial, but stream each step's output as early as possible; do not buffer everything to the end.
- **Hitting rate limits** → lower concurrency + exponential backoff retry + raise quota.

### Core principle

Concurrency raises **throughput**, not **single-request latency**. When a single
response is slow because the model generates token-by-token, concurrency is
ineffective. Use streaming to improve perceived latency; fall back to concurrency
only if the task can be split into independent subtasks.

## Applying

1. Emit one diagnostic line first:
   `诊断：[structure] / [bottleneck] / [method]`
2. Load `references/prompts.md` for the ready-to-use prompt library (a commander
   system prompt + per-scenario execution prompts with `{{placeholders}}`).
3. For implementation, load `references/code_examples.md` for Python (asyncio +
   streaming + connection pool) and JS (fetch streaming) snippets.
4. Execute the chosen method; report wall-clock time and time-to-first-token
   where relevant.

## Pitfalls

- Do not stack concurrency on a compute-bound single request — it yields
  duplicate slow answers and may trip rate limits.
- Streaming does not reduce total generation time; it improves perceived latency.
- Connection pooling only helps repeated calls to the same endpoint.
- Async improves throughput, not the total time of one individual request.
