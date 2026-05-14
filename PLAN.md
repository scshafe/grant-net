# architect-agent — Plan

The resume-point document for the GrantNet architect. When you sit back down at this repo, start here.

## 1. Goal

Stand up a working **Architect agent** that can:

- Read `tenets.md`, `vision.md`, and (as it grows) every spec in `architecture/`, and reason across them coherently.
- Propose, critique, and refine designs for GrantNet's spec layer with fearless ambition (Tenet 1) braked only by feasibility (Tenet 6).
- Hold the bar on the six tenets in `tenets.md` — especially Tenet 1 (fearless conceptual rigor), Tenet 3 (capability discipline), and Tenet 6 (designs must show the path to reality).
- Be invoked by Cole interactively *and* by other sessions via the subagent path (`.claude/agents/architect.md`).

The architect is GrantNet's **bootstrap exception** — every other expert agent (when added) is built either *with* the architect or *to a spec* the architect helped shape. Each hour spent here compounds.

## 2. v0 scope (the minimum that earns its keep)

A Claude Code agent definition that can:

1. **Boot with full project context.** Knows `tenets.md`, `vision.md`, `DESIGN_NOTES.md`, and any `architecture/` content without needing to be told to read each file.
2. **Read tenets and refuse violations.** When asked to design something that contradicts a tenet, says so and explains which tenet — with attention to the scope walls (Tenet 1 vs Tenet 4, Tenet 5 vs the rest).
3. **Write spec drafts, ADRs, and design critiques.** Output is direct — `spec` blocks, `adr` blocks, critique-shaped sections — not just prose.
4. **Run on Cole's machine via Claude Code** — no remote infra needed for v0.

What v0 does *not* need:

- Memory persistence beyond a single Claude Code session.
- Multi-provider runtime (single provider, default Claude; see `DESIGN_NOTES.md` § 1 for seams).
- Auto-invocation from other agents.
- A Structurizr workspace — diagrams come later (see § 6 open question).

## 3. Knowledge surface

See `prompts/system.md` § 1 for the authoritative table.

The architect *reads* but does not *own* the following (in priority order when they conflict):

1. `tenets.md` (binding — *how* we build).
2. `architecture/SCOPE.md` (when it exists — what's in v0).
3. `architecture/concepts.md` (when it exists — vocabulary).
4. `architecture/spec/` (when it exists — system behavior).
5. `architecture/adrs/` (when it exists — decisions).
6. `vision.md` (founding intent; not a substitute for resolved spec).
7. `grant-net-python/` (when it exists — programming model in motion; not authoritative).

## 4. Tools

v0 set:

- `Read` / `Write` / `Edit` / `Bash` (standard Claude Code tools).
- `Grep` / `Glob` for searching across the GrantNet repo.

v1+:

- A `spec_validate` MCP tool for checking spec sections against schema.
- An `adr_lint` MCP tool for ensuring MADR-2.x shape and citation discipline.
- Cross-repo subagent invocation from `grant-net-python/` once that repo exists.

## 5. Milestones

Each milestone is a discrete commit set; *don't* try to land more than one at a time.

### M0 — repo scaffolding (DONE in initial commit)

- README, PLAN, DESIGN_NOTES, tenets.md, vision.md, prompts/{system,style}.md, CLAUDE.md, .claude/agents/architect.md, .gitignore.

### M1 — first dogfood: SCOPE.md

- Architect drafts `architecture/SCOPE.md` bounding v0 scope (per the prompt drafted in the conversation that spawned this repo).
- Goals, Non-goals, In-scope-for-v0, Deferred-to-v1, Open questions, What v0 must demonstrate.
- Verifies: ask "what is in v0 scope?" and the architect answers with citation to SCOPE.md sections.

### M2 — concepts and vocabulary

- Architect drafts `architecture/concepts.md` formalizing grant, tenant, persona, mode, protocol — and the algebra between them.
- This is the document that resolves how persona ⊥ tenant ⊥ mode compose at grant time.
- Verifies: ask "what happens to active grants when a tenant changes persona?" and get a citable answer.

### M3 — foundational ADRs

Land the foundational decisions, in this order (each ADR deferred until the previous is stable):

1. **Capability model.** Token vs OS handle vs cryptographic capability. Threat model implied by each.
2. **Grant lifecycle state machine.** request → approve → use → suspend → revoke. Edge cases.
3. **Multi-tier tenancy model.** What's the principal hierarchy? How do tenants compose?
4. **Persona/mode/protocol composition.** The matching algorithm at grant time.
5. **Pub/sub vs point-to-point cap semantics.** Resolving the known-hard combination.
6. **Backpressure policy taxonomy.** Lossless / lossy / latest-only / bounded-history / batched / blocking.

### M4 — Python prototype scope

Once architecture is firm enough that a Python prototype can be authored against it, scope the Python prototype:

- Which v0 spec sections does it implement?
- What's the API shape (mirroring the systems-language version's interface, per Tenet 4)?
- What's deliberately stubbed / hand-wavy in the prototype's internals (per Tenet 4's scope wall)?
- What feasibility questions does the prototype's existence answer (Tenet 6) — and which does it deliberately defer?

The architect does not write Python — only proposes patches via diff. The Python repo is `grant-net-python/`.

### M5 — eval harness for the architect

A handful of canonical prompts ("draft an ADR for X", "critique this design", "is this Tenet-N-violating?") with expected-shape checks. Catches regressions when prompts or knowledge surface change. Defer until M3 has produced enough material to test against.

## 6. Open questions

- **Diagram tooling.** Structurizr DSL (matches coop-drover's pattern) vs Mermaid in markdown vs C4-PlantUML vs prose-only? Defer the decision until the spec layer needs more than 1-2 diagrams.
- **Spec versioning scheme.** Pre-1.0 evolution rules — how loose is loose enough? Not a tenet question (ABI ceremony is process-rigor, removed) but still a real practical one.
- **Governance.** Cole is sole maintainer for v0. The amendment process for tenets and the role of "trusted contributor" need a doc once external contributors arrive.
- **Public vs internal artifacts.** Will GrantNet have a Patreon-style public vision and a separate internal spec? If so, what's the relationship?

## 7. References

- [`vision.md`](vision.md) — founding intent.
- [`tenets.md`](tenets.md) — the six tenets.
- [`DESIGN_NOTES.md`](DESIGN_NOTES.md) — cross-cutting constraints (provider abstraction, citation discipline, etc.).
- [`README.md`](README.md) — repo overview.
- `prompts/system.md` and `prompts/style.md` — the architect's operating spec.
