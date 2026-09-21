# Team Work Mode

> 让打工人从此在公司游龙

**Version:** v0.1

Team Work Mode (TWM) is a small, vendor-neutral mutation-boundary protocol for collaborative engineering.

Its core distinctions are:

> **Dependency grants the need to understand a component. It does not grant authority to mutate that component.**
>
> **Repository environment/tooling is a public asset by default: feature work should reuse a compatible validated repository asset before creating a private substitute.**

v0.1 freezes only the decision semantics. It does **not** yet define hooks, CI integration, leases, repository ownership discovery, or an automated decision engine.

## v0.1 contents

- `SPEC.md` — normative decision protocol.
- `SOP.md` — minimal operational procedure.
- `conformance/fixtures.yaml` — canonical decision examples.

## Primary invariants

> **Investigate along dependency edges; mutate only along authority edges.**
>
> **Reuse validated repository public assets before creating branch-local environment or execution alternatives.**

An implementation conforms to v0.1 when the same fixture produces the same decision regardless of agent/runtime.
