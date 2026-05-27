# Spec

## Goal

Design the first implementation slice for a Ged-owned workflow orchestrator in
gedcode.

Gedcode should own the workflow lifecycle end to end: classification,
clarification, role invocation, planning artifacts, checkpoint provenance,
guard decisions, verifier invalidation, and eventual commits. Harnesses such as
Codex, Claude, and a future Pi provider should be execution backends for
standalone role threads, not owners of the workflow.

## Users

- A gedcode user running normal chat threads who wants stronger agentic workflow
  guarantees without depending only on prompt instructions.
- Maintainers extending gedcode providers and orchestration without coupling Ged
  workflow behavior to a single harness.

## Current Direction

Add a server-side Ged role invocation/orchestration scaffold that can launch a
single read-oriented `ged-explorer` child thread through existing orchestration
commands. This first slice proves the parent/child thread model and provider
routing without yet intercepting all normal user turns.

The child role thread should:

- be created through the orchestration engine, not by calling provider adapters
  directly;
- copy parent project/worktree/model context where available;
- copy parent `modelSelection`, including its `instanceId`, so routing stays
  provider-instance based without inventing a parallel routing field;
- set `gedWorkflowEnabled: false` to avoid recursive Ged prompt injection;
- create the child thread with a constrained runtime mode such as
  `approval-required` before starting its turn;
- emit parent and child activities so users can see what happened through
  existing timeline surfaces;
- have a stable `invocationId` even before the full durable invocation store is
  built.

## Integration Points

- `packages/contracts/src/orchestration.ts`: existing thread command/event
  surface for thread creation, turn start, activities, checkpoints, and
  projected thread metadata.
- `apps/server/src/orchestration/Layers/OrchestrationEngine.ts`: event-sourced
  command bus. Ged role invocations should dispatch normal commands here.
- `apps/server/src/orchestration/Layers/ProviderCommandReactor.ts`: existing
  bridge from `thread.turn-start-requested` to provider sessions and
  `ProviderService.sendTurn`.
- `apps/server/src/orchestration/Layers/ProviderRuntimeIngestion.ts`: existing
  ingestion path for provider messages, tool events, diffs, plans, activities,
  and runtime failures.
- `apps/server/src/provider/Services/ProviderService.ts`: provider facade. It
  should remain the runtime backend, preferably reached through orchestration.
- `packages/contracts/src/providerInstance.ts`: role invocation routing should
  preserve the parent `ModelSelection.instanceId` instead of hard-coded provider
  kinds.
- `apps/server/src/gedWorkflow/*`: current prompt/guard/checkpoint layer. The
  orchestrator should live here, but child role turns should bypass recursive
  Ged workflow injection.
- `packages/ged-workflow/*`: shared checkpoint and prompt vocabulary. Later
  slices should upgrade this package to gedpi-like v3 checkpoint semantics.
- `apps/web/src/components/chat/WorkflowStatusBadge.tsx` and store/types:
  enough for initial status via existing activities; first-class invocation UI
  can wait.

## Key Decisions

- The first implementation should be explicit or feature-gated. It must not
  automatically listen to every parent `thread.turn-start-requested` yet, because
  that would run the parent provider turn and child role turn in parallel.
- Read-only is provider-dependent in this repo today. The first slice should use
  constrained runtime mode and role prompts as a safety baseline, then defer hard
  provider-level read-only guarantees to later provider capability work.
- Pi should not be the initial core dependency. The orchestrator should be
  provider-agnostic; Pi can become another provider/harness adapter after the
  invocation contract is stable.

## Non-Goals For The First Slice

- Full classify/clarify/plan/verify/commit state machine.
- Automatic interception or replacement of normal parent turns.
- Durable checkpoint schema replacement.
- Parsing explorer output into structured `.ged` artifacts.
- Role-specific model settings UI.
- Pi provider integration.
- Worker execution.
- Child thread cleanup/archive policy.

## Open Questions

- Should role child threads be visible in the sidebar, grouped under the parent,
  or hidden after a durable invocation projection exists?
- Should role provider/model selection initially copy the parent model or use a
  dedicated Ged role setting?
- What is the final output contract for `ged-explorer`: final assistant text,
  JSON, `.ged` artifact, or durable invocation result?
- On parent interrupt, should active role children stop, detach, or finish?
- What restart recovery is required if the server dies after child thread
  creation but before child turn start?

## Acceptance Criteria For Initial Plan Work

- A new branch exists for the design.
- `.ged/work/root/SPEC.md`, `TASKS.md`, and `TESTS.md` describe the initial
  architecture slice.
- The first slice is bounded to explicit/test-only or feature-gated
  `ged-explorer` invocation.
- Planning records identify integration points, risks, tests, and out-of-scope
  work.
