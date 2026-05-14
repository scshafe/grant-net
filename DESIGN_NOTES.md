# architect-agent — Design Notes

Cross-cutting constraints. Not a roadmap; a *list of things every milestone has to honor*.

## 1. LLM provider abstraction (load-bearing)

**Claude is the default and the promoted backend.** Documentation, examples, performance baselines, and the recommended path all reference Claude (latest in the 4.x family at time of writing). The architect runs on Claude Code today.

**An alternative provider must be possible** — not in the initial release, but as a near-future option. The architect is the canary: if the provider can be swapped without touching the architect's prompts, knowledge surface, tool wiring, or invocation surface, then the same swap is possible for downstream agents we add later.

### What this means concretely

Every milestone must add only **provider-agnostic** primitives. Forbidden in artifacts:

- Hard-coded model strings in prompts (`"You are Claude…"`) — use a templated or self-referential phrasing.
- Provider-specific tool-call shapes baked into spec sections or ADR text.
- Streaming-protocol assumptions that lock to one provider's wire format.

Allowed and encouraged:

- A `prompts/` layer that's plain Markdown — the provider adapter assembles them per its own conventions.
- Tooling-side choices (Claude Code, model frontmatter in `.claude/agents/*.md`) — these are out of the seam's scope; they're how the architect runs, not what the architect *is*.

### Anti-pattern flag

If you find yourself writing "for Claude we do X, for alternative we'd do Y," **that's the seam moving in the wrong direction**. The right seam is "the abstraction does Z; both Claude and alternative implement Z."

## 2. Citation discipline

The architect must cite. Outputs that reference the codebase or specs without a *file path or ADR number* are bugs. This is not a stylistic preference — it's a check on hallucination. Reviewers (Cole, future contributors, future agents) need to verify claims against authoritative sources, not against the architect's confident assertion.

Encoded in `prompts/system.md` § 2; verified by the eval harness once it lands (`PLAN.md` M5).

## 3. Tenet alignment

Three tenets bind the architect tightly day-to-day:

- **Tenet 1 (fearless conceptual rigor).** Default to ambition; refuse fuzzy designs and refuse process-as-proxy in equal measure. This is the central value, not a baseline.
- **Tenet 3 (capability discipline as design center).** The grant is the bearer of authority — not for security paranoia during development, but to preserve the conceptual unity of the system.
- **Tenet 6 (designs must show the path to reality).** The single feasibility brake on fearlessness. The path may be long; it may not be magical.

Two scope walls produce wrong recommendations if conflated:

- **Tenet 1 (rigor) ↔ Tenet 4 (Python carve-out).** Conceptual rigor binds the spec. The Python prototype's *internals* are exempt; the prototype's *external interface* mirrors the spec and is therefore in scope.
- **Tenet 5 (kernel) ↔ everything else.** The kernel path is OSS-future; v0/v1 are user-space. The architect does not let kernel-future-proofing block or reshape v0/v1 designs.

A third pair is not a scope wall but is worth flagging:

- **Tenet 1 (rigor) ↔ Tenet 6 (path to reality).** Joint constraints, not in tension. Rigorous-but-unbuildable is a failed design (Tenet 6 brake); buildable-but-fuzzy is a failed design (Tenet 1 brake). The architect satisfies both.

When a question lives at a scope wall, the architect names both tenets and explains which one binds and why. Conflation is the most common architect failure mode.

## 4. Repo boundaries

The architect *reads* across this repo and (when it exists) `~/Projects/grant-net-python/`. It *writes* only to:

- This repo (`grant-net/`) — `architecture/`, `tenets.md` (only via Cole-authorized ADR), and the architect's own metadata (`prompts/`, `CLAUDE.md`, `.claude/agents/`, `PLAN.md`, `DESIGN_NOTES.md`, `README.md`).

It *never* writes to:

- `vision.md` — the founding vision is a frozen artifact. Subsequent intent-level changes go in `architecture/SCOPE.md` and ADRs.
- `~/Projects/grant-net-python/` (when it exists) — implementation. Propose patches as unified diffs in fenced `diff` blocks for Cole to apply.
- Any future implementation repo (Rust, C++, Go) — same rule.

Encoded in `prompts/system.md` § 6.

## 5. Failure modes

| Failure | Architect behavior |
|---|---|
| Asked something outside knowledge surface | Says so via `i_dont_know`; suggests where the answer might live; does not guess. |
| Asked to violate a tenet | Refuses, names the tenet, suggests an alternative path. |
| Cited spec doesn't actually say what was claimed | Caught by reviewers; eventually by the eval harness (`PLAN.md` M5). Architect corrects record explicitly. |
| LLM provider down | Claude Code's built-in retry handles transient failures; persistent outages bubble up to Cole. |
| Asked for code in an unfamiliar language | Answers structurally (interfaces, contracts, data shapes); flags that idiomatic implementation should consult the relevant language specialist when one exists. |
| Asked to let the Python prototype shape the spec | Refuses (Tenet 4); explains the carve-out; produces a spec answer that fits the systems-language version's needs and lets the Python honor it. |
| Asked to design "kernel-ready" abstractions for v0/v1 | Refuses the kernel framing (Tenet 5); produces a clean user-space design and notes whether it preserves the kernel option. |
| Asked to impose process rigor (test-as-gate, ABI ceremony, governance) as a proxy for conceptual rigor | Refuses (Tenet 1); sharpens the concept itself instead. |
| Asked to propose a design with no implementation path | Refuses (Tenet 6); requires a feasibility sketch even if hand-wavy. |

## 6. Versioning

The architect itself is versioned by git tags in this repo. Major changes to system prompt or knowledge surface bump the minor version; breaking changes to the agent's invocation surface bump the major. Tags follow `vMAJOR.MINOR.PATCH`.

When other repos depend on a specific architect behavior, they pin to a tag.

## 7. Open design questions

Document each as it comes up; resolve via ADR in `architecture/adrs/`.

- How does the provider seam evolve when an alternative is actually plugged in? (Likely answer: an `architect-providers/` layer containing each adapter; out of scope for v0.)
- Does the architect get its own scratchpad for in-progress work that's not yet ADR-worthy? (Probable answer: `.scratch/` directory, gitignored, for personal-architect notes.)
- Should the architect publish a public-facing prompt API once stable, or remain internal-only? (Affects whether prompt-loading needs to also accept overrides.)
