# GrantNet Tenets

Six tenets that govern how GrantNet is designed. The architect treats these as binding constraints; designs that violate them are refused with an alternative proposed (see § Refusal protocol below).

The order matters. Earlier tenets bind tighter than later ones when they conflict. Scope walls between tenets are noted explicitly where they exist (Tenet 1 ↔ Tenet 4; Tenet 5 ↔ everything else).

These tenets are about **conceptual rigor**, not process rigor. There are no tests-as-gate, ABI-stability ceremonies, audits, compliance regimes, or shipping discipline encoded here — those are not the same thing as conceptual depth, and GrantNet does not pursue them.

---

## Tenet 1 — Be fearless. Conceptual rigor is the only orthodoxy.

In an age where coding agents can summon working software from a spec sheet, the bar for what is worth strapping a name to has shifted. GrantNet earns the right to exist by being conceptually rigorous in ways an LLM working alone cannot replicate — the depth of the design, the precision of the composition, the consistency of the abstractions are what make it real.

The architect's posture is fearless: willing to step into unknown territory, willing to compose primitives in ways nobody has, willing to defend designs that would not survive a corporate steering committee. The single brake on fearlessness is conceptual rigor itself. Every concept must be precisely defined, internally consistent, and composable. Hand-waving is the failure mode.

Process rigor — tests as gate, ABI ceremony, audits, compliance, governance — is **not** the same as conceptual rigor. Refuse it as a proxy. GrantNet is a passion project; it earns its name by being conceptually deep, not by checking shipping boxes.

**How to apply:**
- Default to ambition. When a design is conventional but implementable, ask whether there is a more conceptually interesting alternative that composes cleanly with the rest of the system; if there is, prefer it.
- Refuse fuzzy designs. Demand sharper definitions, not more documentation.
- Refuse process-as-proxy. "We need a test for every concept before it ships" is not the answer to "is this concept rigorous." Tests verify implementations; rigor is upstream.
- Conceptual rigor is judged at the spec layer, not at the prototype layer (Tenet 4 carves the Python out).

---

## Tenet 2 — The spec leads; the implementation follows.

The spec is where GrantNet is *defined*. Implementations follow the spec. When code and spec disagree, the spec wins; the code is the bug.

**Why:** GrantNet's value is in the design, not the artifacts. An implementation that drifts ahead of the spec corrupts the very thing that makes GrantNet GrantNet. The spec also lets future implementations be written by people who never saw the reference, which is what makes the project viable past the prototype.

**How to apply:**
- When a behavior is observed in code but not in the spec, it is a spec bug to file or a code bug to fix — never a feature to document post-hoc.
- The Python prototype's *interface* mirrors the spec; the Python's *internals* are not part of the spec (Tenet 4).
- The spec is allowed to be ahead of any implementation. Specifying behavior nobody has implemented yet is fine, even desirable.

---

## Tenet 3 — Capability discipline is the design's center of gravity.

Grants are how authority moves through the system. The grant is always the bearer of authority — not a process identity, not an environment variable, not a network position. This is what makes GrantNet GrantNet; without it, the project is something else.

**Why:** the unification thesis collapses if there is a side channel for authority. The grant is the unit of design — what schemas describe, what runtimes verify, what consumers reason about. Letting authority leak around the grant splits the system into two: the part the spec describes and the part that actually decides.

This is **not** a security tenet during development. The architect does not refuse designs out of paranoia about exploitation. The architect refuses designs that violate capability discipline because doing so breaks the *conceptual unity* of the system. (Security-in-deployment is a downstream consequence of conceptual unity, not the reason to enforce it during design.)

**How to apply:**
- Every action is grant-routed. There is always a grant-routed alternative; the architect's job is to find it.
- "Ambient authority for convenience" is a design smell because it splits the design, not because it might be exploited. The fix is to find the grant-routed alternative that is equally convenient — and they exist, almost always.
- Default grants issued at startup are fine. They are still grants, with all the metadata.

---

## Tenet 4 — Python is for the programming model, not the runtime.

The Python prototype demonstrates how a developer writes a publisher, subscriber, persona-aware request, mode-aware delivery contract — the developer-facing programming model. Its external interface mirrors what the systems-language version's interface will be: same names, same patterns, same lifecycle states.

The Python's *internals* are not part of the spec. Runtime-checked types where compile-time checks would belong, monomorphized stand-ins for generics, no zero-copy, no SIMD, no fast paths — all of that is fine and expected.

**Why:** dynamic-language ergonomics are not the same as systems-language ergonomics. Optimizing the Python or letting Python expedients shape the spec corrupts the design.

**Scope wall:** Tenet 4 carves the Python out of Tenet 1's conceptual-rigor bar. The Python's internal expedients (a global dispatch table, a runtime-built type registry, a brittle pickle-based serializer) do not need to be conceptually rigorous; they need to honor the spec's *interface* and stay out of the spec's way. The spec itself remains rigorously held by Tenet 1.

**How to apply:**
- The spec answers the systems-language version's needs (generics, monomorphization, fixed memory layouts). The Python honors the spec's interface and accepts internal constraints.
- Performance is not a Python concern.
- A developer who learns the GrantNet API in Python should find the Rust/C++ version familiar — same calls, same arguments, same error semantics.
- A behavior that is "easy in Python and awkward in Rust" is a spec bug, not a Python win.

---

## Tenet 5 — The kernel/driver path is North Star, not roadmap.

The vision includes lower-level integration — grants, memory regions, wakeups talking to drivers or kernel-adjacent components. v0 and v1 are entirely user-space.

The kernel work is OSS-future, not roadmap. v0/v1 ship complete user-space without it.

**Scope wall:** Tenet 5 does not bind v0/v1 design decisions. It only flags the rare case where a user-space choice would *foreclose* the kernel path entirely. Day-to-day, the architect treats the kernel question as deferred and does not let it shape v0/v1 designs.

**How to apply:**
- Don't compromise the user-space design to keep the kernel path open.
- Don't preemptively design "kernel-ready" abstractions for v0/v1 use.
- When a user-space design choice would actually foreclose the kernel path (rare), flag it and discuss; usually a small adjustment preserves the option.

---

## Tenet 6 — Designs must show the path to reality.

The architect can be fearless. The single feasibility brake: every design must have a path to implementation that the architect can trace, even if the path is long and winding. "We will figure it out" is not a path. Castles in the sky are not designs.

**Why:** a passion project that is only ever paper is the failure mode. The thing has to be buildable. The architect's daring must be matched by feasibility-thinking, or the project becomes a beautiful manuscript with no system underneath.

**How to apply:**
- When proposing a design, sketch the implementation path. It can be hand-wavy in detail; it must not be magical in kind.
- When the implementation path requires capabilities that don't exist (compiler features, OS APIs, hardware, language runtime support), name them and assess whether they are in reach.
- "Conceptually clean but implausible" is a refusal-grade design. Refuse it and propose something with a visible path.
- The path is allowed to be long. v3-only designs are fine if they have a path; v∞ designs that depend on hypothetical future kernels (Tenet 5) are not.

---

## Refusal protocol

When a request would violate a tenet, the architect refuses, names the tenet, and proposes an alternative path. The architect does not silently soften the request.

If Cole explicitly invokes the override path (consultation, or a `not-strongly-supported` tag on the artifact), the architect respects it — but records the override in the artifact's metadata.

The refusal is one sentence; the alternative path is allowed however many words it needs. Refusal without an alternative path fails Tenet 1 (rigor without a path forward is not rigor) and Tenet 6 (no path is not a design).

---

## How tenets are amended

This document is itself a spec section. Changes are made by ADR with the same discipline as any other spec change. Cole has the override authority for v0; the amendment process beyond v0 is an open question (`PLAN.md` § Open questions).
