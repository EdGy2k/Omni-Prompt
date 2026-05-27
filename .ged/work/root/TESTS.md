# Tests

## Plan

No source implementation has started yet. For the initial implementation slice,
use focused tests before the required repo-wide checks.

## Initial Slice Test Coverage

- Service test: invoking `ged-explorer` creates a child thread with copied
  project/worktree/model context and `gedWorkflowEnabled: false`.
- Activity test: parent receives an explorer-started activity; child receives a
  role-child activity referencing the parent and invocation id.
- Provider reactor integration test: child `thread.turn.start` routes through
  `ProviderInstanceId` and calls `sendTurn` with `gedWorkflowEnabled: false`.
- Safety test: child uses constrained runtime mode instead of inheriting a
  full-access parent mode.
- Prompt test: explorer prompt states read-only role boundaries and required
  output shape.
- Later durable-state test: repeated invocation id does not create duplicate
  child threads once invocation persistence exists.

## Required Completion Checks

- `bun fmt`
- `bun lint`
- `bun typecheck`
- Targeted `bun run test ...` suites for changed server/contracts/web packages.

## Evidence

- `bun fmt` passed: oxfmt completed on 1123 files.
- `bun lint` passed with 0 errors and 10 existing warnings.
- `bun typecheck` passed across all 14 Turbo packages.
- Read-only verifier reviewed the final planning diff and found no blockers.
- Source implementation is pending user approval of the initial design.
