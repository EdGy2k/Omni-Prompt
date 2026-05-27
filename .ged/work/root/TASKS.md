# Tasks

## ged-owned-workflow-orchestrator-design

1. [x] Create branch `feat/ged-owned-workflow-orchestrator` from `origin/main`.
2. [x] Run read-only explorer discovery for current provider/orchestration/Ged
       integration points.
3. [x] Run read-only planner critique for first-slice sequencing, risks, and
       tests.
4. [x] Replace stale `.ged/work/root` planning artifacts with the initial
       orchestrator design.
5. [ ] Review the plan with the user before source implementation.

## Initial Implementation Slice: `GedRoleInvocationService`

1. [ ] Add a small server-side service under `apps/server/src/gedWorkflow`.
2. [ ] Define an initial `GedRoleInvocationInput` for one role:
       `ged-explorer`.
3. [ ] Generate or accept a stable `invocationId`.
4. [ ] Resolve parent thread/project/worktree/model context from existing
       projections or command inputs.
5. [ ] Dispatch normal orchestration commands:
   - create child thread;
   - append parent "ged-explorer started" activity;
   - append child "role child of parent" activity;
   - start child turn with role prompt.
6. [ ] Ensure child turn uses:
   - copied parent `modelSelection`, including `instanceId`;
   - copied project/worktree context;
   - `gedWorkflowEnabled: false`;
   - child thread created with constrained runtime mode such as
     `approval-required` before `thread.turn.start`.
7. [ ] Keep invocation explicit/test-only or feature-gated.
8. [ ] Add focused tests for child thread creation, activity emission, and
       provider turn dispatch inputs.

## Follow-Up Slices

1. [ ] Add a durable `GedWorkflowInvocation` contract/projection.
2. [ ] Add idempotency and restart recovery around `invocationId`.
3. [ ] Add role completion tracking and checkpoint auto-recording.
4. [ ] Upgrade `packages/ged-workflow` to gedpi-compatible v3 semantics:
       clarification, explorer, planner, plan acceptance, verifier, worker audit,
       lifecycle closure, fallback provenance.
5. [ ] Add parent-turn gating so Ged-owned workflow can replace normal provider
       dispatch for workflow-managed turns.
6. [ ] Add first-class web workflow timeline/invocation status.
7. [ ] Add Pi provider/harness integration after the provider-agnostic contract
       is stable.
