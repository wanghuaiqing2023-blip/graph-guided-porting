---
name: graph-guided-porting
description: Guide project-to-project language ports using a dependency knowledge graph as the migration map. Use when Codex needs to port, rewrite, transpile, or reimplement an existing codebase in another programming language, especially when a knowledge graph such as `.understand-anything/knowledge-graph.json` is available and the work should preserve behavior through graph-selected end-to-end slices instead of file-by-file translation.
---

# Graph Guided Porting

## Overview

Use the source project's knowledge graph as a navigation index for behavior-preserving language ports. Prefer runnable, user-facing dependency slices over mechanical file-by-file translation.

## Core Rule

Treat the graph as a map, not the source of truth:

1. Use the graph to identify relevant files, symbols, relationships, entrypoints, and high-impact nodes.
2. Read the authoritative source code for the selected slice before deciding behavior.
3. Implement the smallest coherent target-language slice that advances an end-to-end user-facing flow.
4. Validate touched behavior, then document progress and known gaps.

## Workflow

### 1. Establish Scope

Identify:

- Source language and target language.
- Source root and target root.
- Knowledge graph path.
- User-facing entrypoint or behavior being ported.
- Explicitly excluded extension areas.

If scope is not supplied, infer a conservative core-path slice from the repository context and user request.

### 2. Query The Graph Selectively

Do not load the whole graph into context. Use small scripts, `jq`, or targeted JSON inspection to answer specific questions:

- Which nodes represent the requested entrypoint?
- Which files contain those nodes?
- Which `calls`, `imports`, `exports`, or containment edges connect them?
- Which nearby nodes have high fan-in or fan-out?
- Which branches lead into peripheral systems that can be shimmed or deferred?

Summarize only the relevant nodes, files, and relationships.

### 3. Cut A Dependency Slice

Start from a user-facing entrypoint and follow graph relationships until the minimal behavior path is visible.

Prefer slices like:

- CLI command -> config -> context assembly -> request construction -> streaming -> final answer.
- Agent loop -> tool-call parsing -> tool dispatch -> shell/file/patch execution -> tool result.
- Session loading -> conversation state -> model request items -> response handling.

Avoid starting from leaf helpers unless they unblock the current slice.

### 4. Rank Work

Prioritize in this order:

1. End-to-end runnable core flows.
2. High fan-in or high fan-out nodes on the selected core path.
3. Shared data contracts and event/result types needed by multiple core modules.
4. Boundary parity and focused tests for the selected slice.
5. Compatibility shims for peripheral systems.

Defer deep implementation of plugin, marketplace, MCP, cloud, telemetry, and daemon branches unless the user explicitly asks or the selected core path truly depends on them.

### 5. Read Source, Then Port Behavior

For each selected node or file:

- Read the smallest authoritative source files needed to understand behavior.
- Capture externally visible behavior, data shapes, errors, ordering, and side effects.
- Map behavior to idiomatic target-language modules without blindly copying source layout.
- Prefer standard-library or already-approved dependencies in the target project.
- Keep compatibility shims explicit and documented.

### 6. Validate The Slice

Validate only the touched behavior unless broader validation is requested.

Useful checks:

- Unit tests for data conversion, policy decisions, parsing, and tool result handling.
- End-to-end smoke tests for CLI or runtime slices.
- Golden-output or fixture comparisons where the source behavior is deterministic.
- Manual command runs when tests do not yet exist.

If validation is blocked, report the blocker and the residual risk.

### 7. Record Progress

For meaningful progress, update the project's porting status or notes if such files exist.

Document:

- Source graph nodes/files used.
- Source behavior confirmed from code.
- Target modules changed.
- Validation run.
- Known gaps and deferred branches.

## Decision Heuristics

- Prefer graph-selected mainline progress over isolated local edge cases.
- Prefer an imperfect but runnable core slice over many precise helpers that are not connected.
- Prefer shims over deep peripheral ports for extension systems outside the active objective.
- Prefer behavior parity over source-layout parity.
- Prefer targeted graph queries over broad source searches.
- Use broad search only after the graph fails to locate the relevant code.

## Typical Output Shape

When reporting a plan or result, include:

- Selected entrypoint or behavior.
- Relevant source graph slice.
- Target-language implementation slice.
- Validation strategy or completed checks.
- Deferred branches or compatibility shims.

