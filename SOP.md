# Team Work Mode SOP v0.1

This SOP applies the protocol without adding new authority rules.

## Repository public asset gate

Before creating or installing a new environment, dependency setup, container, build path, or local workaround:

1. Classify the failure: source/behavior problem or environment/execution problem.
2. Inspect repository public assets already validated by the current repository: Docker/DevContainer definitions, CI images, build scripts, dependency bootstrap scripts, and the environment used by newer/higher-level branches or the default branch.
3. If a compatible public asset exists, reuse it. Do **not** create a branch-private substitute merely because the current host/worktree lacks a dependency.
4. Treat `build/`, `install/`, generated code, caches, and binaries as source-state-derived assets. Regenerate them for the current commit/worktree unless compatibility is proven.
5. Only create a new environment/execution path when no compatible validated repository public asset can satisfy the claim.

## Before mutation

1. State the current `claim` in one sentence.
2. Identify the proposed `target`.
3. Ask: **Is changing this target required for the claim?**
4. Ask: **Does the current claim have mutation authority over this target?**
5. Identify any frozen/forbidden delta that must remain unchanged.
6. Run the v0.1 decision order from `SPEC.md`.

## Execute by decision

- `ALLOW` — make the smallest mutation needed for the claim.
- `RECORD_ONLY` — record the issue; do not fix it in this claim.
- `HANDOFF` — stop mutation at the ownership boundary and provide the blocking evidence.
- `DENY` — preserve the frozen contract and choose a different implementation path.
- `REQUIRE_GATE` — send the proposal to the coordinator/main gate; do not accept it automatically.
- `BOUNDARY_RECHECK` — stop patch accumulation and re-check whether the historical ownership/decomposition is wrong.

## Verification

Verify at the same level as the claim:

- file/function claim -> focused checks may be sufficient;
- module claim -> module tests/build;
- cross-repository claim -> cross-repository contract/integration verification;
- user-visible/system claim -> end-to-end evidence.

A green lower-level check MUST NOT be reported as proof of a higher-level claim.
