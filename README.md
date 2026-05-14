# grant-net

The architecture and specification home for **GrantNet** — a typed capability IPC substrate for tenant-scoped, persona/mode-aware, JIT-authorized, high-performance zero-copy local streams.

This repo holds the spec, the tenets, the architect agent that produces them, and the project's founding vision. It does not hold any implementation code; that lives in sibling repos (`grant-net-python` for the Python prototype; future systems-language ports as separate repos).

## Status

Pre-spec. The architect agent is scaffolded (this commit). The first architectural deliverable is `architecture/SCOPE.md`, drafted by the architect.

## What lives here

```
grant-net/
  README.md                     — this file
  vision.md                     — founding vision (frozen)
  tenets.md                     — the six tenets that govern design
  PLAN.md                       — the architect agent's milestones and resume-point
  DESIGN_NOTES.md               — cross-cutting constraints (provider abstraction, citation, scope walls)

  CLAUDE.md                     — wires the architect into Claude Code: a session in this repo *is* the architect
  .claude/agents/architect.md   — exposes the architect as a subagent for cross-repo invocation
  prompts/
    system.md                   — the architect's system prompt
    style.md                    — the architect's voice and formatting

  architecture/                 — the spec layer (drafted by the architect over time)
    SCOPE.md                      what's in v0, what's deferred
    concepts.md                   vocabulary and the algebra of grants/tenants/personas/modes/protocols
    spec/                         the spec layer proper — protocol formats, grant lifecycle, runtime semantics
    adrs/                         accepted/proposed decisions (MADR-2.x format)
    diagrams/                     Structurizr DSL or equivalent (TBD; see PLAN.md § Open questions)
```

The `architecture/` subtree starts empty; the architect populates it as it produces artifacts.

## Sibling repos

- **`grant-net-python`** — the Python prototype. Demonstrates the programming model. Not authoritative on what the system is; the spec is. Per `tenets.md` Tenet 1.

Future systems-language ports (Rust / C / C++ / Zig) will live in their own repos as they're authored.

## Working with the architect

Two ways to invoke the architect:

- **Direct mode.** `cd ~/Projects/grant-net && claude`. The session loads the architect persona via `CLAUDE.md` and is the architect from turn 1. Use this for spec drafting, ADR work, design critiques.
- **Subagent mode.** From any sibling repo's Claude Code session, dispatch via the Agent tool with `subagent_type: architect`. Claude Code finds the agent definition by walking up to `grant-net/.claude/agents/`. Use this when you want a parent session to consult the architect for a focused question without losing its own context.

The architect's contracts (`prompts/system.md` § 4) define the artifact shapes it produces — ADR drafts, spec drafts, design critiques, optional Structurizr DSL fragments. Each output ends with a confidence tag (`§ 4.5`).

## Project values

The six tenets in `tenets.md` are the binding constraints on design. They are about **conceptual rigor**, not process rigor — there are no tests-as-gate, ABI ceremonies, audits, compliance regimes, or shipping discipline encoded. GrantNet is a passion project; it earns its name by being conceptually deep, not by checking shipping boxes.

Three tenets to know up front:

- **Tenet 1 — Be fearless. Conceptual rigor is the only orthodoxy.** Default to ambition. The single brake on fearlessness is conceptual rigor itself: precise definitions, internal consistency, clean composition. Process rigor is not a substitute.
- **Tenet 3 — Capability discipline is the design's center of gravity.** The grant is always the bearer of authority. This is what makes GrantNet GrantNet — without it, the project is something else.
- **Tenet 6 — Designs must show the path to reality.** Fearless ambition is constrained by feasibility. Every design must have a path to implementation that the architect can trace, even if long and winding. Castles in the sky are not designs.

The full set lives in `tenets.md`.

## License

TBD. (To be selected before the first public release; the architect can produce an ADR scoping the choice.)
