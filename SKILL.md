---
name: tool-design
description: Guides naming, consolidation, and output design for agent-facing tools (MCP, function calling, internal APIs). Use when designing new tools, reviewing tool sets, tightening payloads, or when the user mentions agent tools, MCP tools, or tool schemas.
disable-model-invocation: true
---

# Tool design

Principles for tools that humans and LLM agents call. Applies to MCP tools, function-calling definitions, and similar interfaces.

## 1. Names must state the goal

The name of the file and the name of the tool must be easy and intuitive for both humans and AI agents to understand the clear goal of it. For example: let's suppose I'm creating a tool for our Customer Service agent that lets a customer log in into their account. A bad name would be `get_user_by_document`. A clear, more semantically correct name would be `get_customer`.

**Check:** Can a new reader guess *what success looks like* from the name alone? Prefer domain language (`customer`, `order`) over implementation (`document`, `row`).

## 2. Prefer fewer, richer tools

Less tools are better than many tools. If the goal is, for example, to diagnose a bad Internet connection, instead of having three tools for the job, like `check_major_outage`, `check_onu_signal`, `check_router_electricity`, etc. It's better to compress every step in an early-return designed and unique `check_connection_health` tool.

**Pattern:** One tool per user- or agent-facing *job*. Orchestrate sub-steps inside the implementation; return as soon as the answer is known (early returns). Split only when callers genuinely need independent operations or different auth/scope.

## 3. Results are user-facing, not debug dumps

A tool must always return clear, user-friendly, and intuitive results. It must not leak internal system workings or complex, technical terms. Example of a bad result: `{ "error": 504: TypeError in line X of file Y }`. Example of an acceptable result: `{ "error": "We couldn't connect to the server. Try again in X minutes" }`.

**Do:** Stable shape (`ok` / `error`, or domain-specific fields), human-readable messages, safe correlation IDs if support needs them (opaque, not stack traces).

**Don’t:** Stack traces, file paths, raw framework errors, or internal service names unless the audience is strictly engineers *and* the spec says so.

## 4. Minimize payload size

Tool results must be as shorter as possible, since each token counts, and they accumulate over time with every request. Provide only the strictly needed information.

**Do:** Omit nulls where possible; use short, consistent keys; return lists summaries instead of huge nested blobs when the agent only needs a decision.

**Don’t:** Echo full request objects, redundant duplicates, or verbose explanations in every field.

## 5. Implementation discipline

General Good practices: apply KISS, DRY, and Separation of Concerns when developing a tool. Use as many explanatory comments as needed so each block is crystal-clear for a human developer to read.

**Structure suggestion:** Thin handler (validate input → call domain/service → map to response). Shared validation and mapping live in one place (DRY). Side effects and pure logic separated (Separation of Concerns).

## Quick checklist

- [ ] Name reflects the *goal* in domain vocabulary
- [ ] Tool count minimized; early-return flow inside one job-shaped entrypoint where it fits
- [ ] Responses short, stable schema, friendly errors, no internal leakage
- [ ] Only fields the agent needs next turn
- [ ] Code KISS/DRY/SoC; comments for non-obvious blocks
