# Team Work Mode Protocol v0.1

Status: Draft normative specification.

Normative keywords **MUST**, **MUST NOT**, **SHOULD**, and **MAY** are used in their ordinary RFC-style sense.

## 1. Model

For every proposed mutation, TWM distinguishes two relations:

- **Dependency** — component A needs component B to satisfy A's claim.
- **Mutation authority** — the current claim is authorized to change B's behavior or contract.

These relations MUST NOT be treated as equivalent.

TWM also distinguishes **repository public assets** from branch-local derived state:

- **Repository public assets** — validated environment and execution definitions shared by the repository, such as Docker/DevContainer images, CI images, build scripts, dependency bootstrap scripts, and repository-standard toolchain configuration. A compatible public asset MAY come from the current branch, a newer/higher-level branch, the default branch, or CI.
- **Derived assets** — outputs tied to a concrete source state, such as `build/`, `install/`, generated code, caches, and compiled artifacts. These MUST NOT be assumed compatible across commits or worktrees without evidence.

> **TWM-1 — Dependency MUST NOT imply mutation authority.**

The protocol evaluates a proposed mutation using:

- `claim`: the observable result currently being delivered.
- `target`: the component/contract proposed for mutation.
- `required`: whether changing the target is necessary for the current claim.
- `authority`: whether the current claim owns or is explicitly authorized to mutate the target.
- `forbidden_delta`: behavior/contracts explicitly required to remain unchanged.
- `source`: who proposed the mutation, e.g. executor, reviewer, audit.
- `coordinator_accepted`: whether advisory review output has passed the coordinator/main gate.
- `repeated_boundary_patching`: whether multiple adjacent fixes indicate the current decomposition may be wrong.

## 2. Normative rules

**TWM-2 — Every mutation MUST be attributable to one current claim and one mutation authority.**

A mutation without an identified claim or authority MUST NOT proceed.

**TWM-3 — A discovered issue that is not required for the current claim MUST NOT expand mutation scope.**

It MAY be recorded for later work.

**TWM-4 — A required issue outside current mutation authority MUST transition to HANDOFF.**

The executor MAY inspect enough dependency context to identify the blocking boundary, but MUST NOT absorb authority implicitly.

**TWM-5 — A proposed mutation that violates a frozen/forbidden delta MUST be denied even when all affected components can be made internally consistent.**

**TWM-6 — Review or audit output is advisory evidence, not mutation authority.**

A mutation proposed only by a reviewer/auditor MUST pass the coordinator/main gate before it becomes executable work.

**TWM-7 — Repeated adjacent patching across a boundary MUST trigger a boundary re-check before further mutation.**

Progress pressure MUST NOT be used as justification for continuing mutation.

**TWM-8 — Verification MUST match the claim level.**

Local/unit success MUST NOT be represented as proof of cross-repository or end-to-end correctness.

**TWM-9 — Repository public assets MUST be checked before creating a private environment or execution path.**

When a claim encounters a missing dependency, tool, build environment, container, or execution path, the executor MUST first inspect the repository's already-validated public assets. If a compatible public asset exists, the executor MUST reuse it instead of constructing an ad-hoc branch-local substitute. A missing host dependency by itself does not make private environment construction `required`.

Reusing a public asset does not grant mutation authority over that asset. Derived assets from another commit/worktree MUST be regenerated for the current source state unless compatibility is separately proven.

## 3. Decision states

A conforming evaluator MUST return exactly one primary decision:

- `ALLOW` — required by current claim and within current mutation authority.
- `RECORD_ONLY` — discovered issue is not required for current claim.
- `HANDOFF` — required, but target is outside current mutation authority or authority is unresolved.
- `DENY` — proposed mutation violates an explicit forbidden delta or frozen contract.
- `REQUIRE_GATE` — mutation originates from advisory review/audit and lacks coordinator acceptance.
- `BOUNDARY_RECHECK` — repeated adjacent patching indicates that ownership/decomposition must be re-evaluated before more mutation.

## 4. Deterministic decision order

When multiple conditions are true, evaluators MUST use this precedence:

```text
1. forbidden_delta violated
      -> DENY

2. repeated_boundary_patching
      -> BOUNDARY_RECHECK

3. source is reviewer/audit AND coordinator_accepted is false
      -> REQUIRE_GATE

4. required is false
      -> RECORD_ONLY

5. authority is false or unresolved
      -> HANDOFF

6. otherwise
      -> ALLOW
```

This precedence prevents a lower-level authorization from overriding a higher-level scope or governance violation.

The repository-public-asset gate is a **pre-evaluation normalization**, not a new primary decision state. For an environment/tooling mutation, if a compatible validated repository public asset already satisfies the claim, the proposed private replacement is `required = false` and therefore resolves through the existing decision order (normally `RECORD_ONLY`).

## 5. Decision tree

```text
                 Proposed mutation
                        |
          Violates frozen/forbidden delta?
                 /              \
               yes              no
               |                 |
             DENY       Repeated boundary patches?
                           /             \
                         yes             no
                         |                |
                 BOUNDARY_RECHECK   Review-only proposal
                                         without gate?
                                       /           \
                                     yes           no
                                     |              |
                               REQUIRE_GATE    Required by claim?
                                               /          \
                                             no           yes
                                             |             |
                                       RECORD_ONLY     Authorized owner?
                                                       /          \
                                                     no           yes
                                                     |             |
                                                  HANDOFF        ALLOW
```

## 6. Non-goals of v0.1

v0.1 does not define:

- how repository ownership is discovered;
- how claims are leased or persisted;
- how GitHub/GitLab permissions are enforced;
- how a coordinator is selected;
- automatic mutation blocking;
- automated discovery/ranking of repository public assets;
- agent-specific prompts or hooks.

Those are adapters/policy concerns and MUST NOT redefine the protocol decisions above.
