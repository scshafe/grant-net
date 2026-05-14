# GrantNet v0 Scope

* **Status:** draft
* **Tenets bound:** Tenet 1, Tenet 2, Tenet 3, Tenet 5, Tenet 6 (`tenets.md`)
* **Last reviewed:** 2026-05-08
* **Scope stance:** v0 is a vertical grant-algebra slice, not a flat subset of `vision.md`.
* **Overall confidence:** medium - the scope is grounded in `tenets.md`, `vision.md`, `PLAN.md`, and prior art; v0 epoch semantics are now explicit, but mode-change mechanics and first-implementation language choice need ADR confirmation.

## Scope stance

A flat subset of `vision.md` concerns is the wrong shape for v0. It would invite a feature checklist: typed protocols, then grants, then tenants, then modes. That proves components exist. It does not prove GrantNet's unification thesis.

v0 instead commits to the smallest vertical slice where every operation is grant-routed and every visible behavior can be explained by the same grant record:

* `protocol_id`
* `rights`
* `tenant`
* `persona`
* `mode`
* `delivery_policy`
* `resource_limits`
* `lifecycle_state`
* `epoch`

That slice is ambitious because it composes typed IPC, authorization, tenancy, persona, mode, backpressure, wake behavior, resource limits, and revocation into one contract. It is constrained because it avoids kernel work, distributed networking, production kernel integration, dynamic protocol loading, durable capabilities, and process ceremony (`tenets.md` Tenet 1, Tenet 3, Tenet 5, Tenet 6; `vision.md` "Core features").

## Layer model

| Layer | v0 meaning | Scope wall |
|---|---|---|
| Spec v0 | Normative GrantNet behavior: concepts, grant fields, lifecycle states, delivery semantics, error semantics, and the developer-facing API contract. | The spec leads the implementation (`tenets.md` Tenet 2). |
| First implementation v0 | A user-space systems-language implementation that exercises the real ownership, layout, and grant-table shape. C++ is the current candidate because it can test the later kernel-adjacent path without making kernel work part of v0. | The path must be plausible and user-space (`tenets.md` Tenet 5; `tenets.md` Tenet 6). |
| Portability path | A visible route to future Rust/C/Zig bindings or ports with the same API concepts, lifecycle states, and error semantics. | The spec must define GrantNet, not the first implementation's local idioms (`tenets.md` Tenet 2). |
| Kernel/driver path | Not v0. Not v1. Only avoid choices that foreclose a later kernel path. | Kernel and driver integration are OSS-future (`tenets.md` Tenet 5; `vision.md` "Kernel/driver integration path"). |

## Goals

1. **Demonstrate that a grant can be the single authority and delivery contract.** v0 must show that opening, publishing, subscribing, receiving, downgrading, suspending, and revoking all route through a grant, not a process identity, environment variable, service-manager lookup, or raw topic string (`tenets.md` Tenet 3).

2. **Make protocol definitions carry more than payload shape.** A v0 protocol defines typed payloads, rights, stream kind, delivery policies, compatible personas, compatible modes, resource-limit dimensions, and revocation behavior. This is where GrantNet diverges from IDLs that define message structure but leave policy outside the protocol (`vision.md` "First-class typed protocols").

3. **Prove tenant/persona/mode composition at grant time.** v0 must show a request from `tenant T` under `persona P` in `mode M` producing a grant whose rights and delivery behavior are narrower than the raw protocol's full capability (`vision.md` "Multi-tier multi-tenancy", "Persona-aware access", "Mode-aware delivery").

4. **Show JIT authorization without ambient authority.** v0 must support approve, deny, downgrade, expire, suspend, and revoke outcomes on grant requests. The first implementation may use a simple in-process authorizer, but the behavior belongs to the spec (`vision.md` "JIT authorization"; `tenets.md` Tenet 3).

5. **Show pub/sub fanout and latest-state through the same grant algebra.** v0 must include both event stream delivery and blackboard/latest-state delivery because the unification thesis needs one-to-many history and state replacement, not one of them alone (`vision.md` "Publish/subscribe and fanout support", "Blackboard/latest-state model").

6. **Exercise the zero-copy path in the first implementation shape.** v0 spec includes sample ownership and payload-layout eligibility. The first implementation should use APIs that can map to loaned shared-memory chunks, even if the first transport uses a simpler user-space backing store (`vision.md` "High-performance zero-copy local streams"; `tenets.md` Tenet 6). **Confidence: medium - the API shape is plausible, but the exact ownership vocabulary needs `architecture/concepts.md` and an ADR.**

7. **Refuse process rigor as scope.** v0 is not "done" because tests, ABI rules, governance, audits, or release ceremony exist. v0 is real when its concepts are sharp and its smallest programs demonstrate them (`tenets.md` Tenet 1; `PLAN.md` M1).

## Non-goals

| Non-goal | Layer call | Reason |
|---|---|---|
| Test-as-gate, ABI stability ceremony, governance process, audit/compliance posture, release maturity | Out of spec v0 and implementation v0 | These are process-rigor substitutes, not conceptual rigor (`tenets.md` Tenet 1). |
| Kernel objects, driver integration, kernel wakeups, kernel-mediated grants, custom syscalls | Out of v0 and v1 | The kernel path is OSS-future (`tenets.md` Tenet 5; `vision.md` "Kernel/driver integration path"). |
| Production zero-copy performance, throughput targets, latency targets, SIMD, lock-free internals | Out of v0 acceptance | v0 should test the real ownership model, but performance targets are not the proof of conceptual rigor (`tenets.md` Tenet 1; `tenets.md` Tenet 6). |
| Kernel-grade shared-memory allocator, driver-facing memory regions, and true zero-copy fanout | Out of v0 implementation; spec v0 keeps ownership shape and layout eligibility | The future shared-memory path needs this semantic hook, but kernel-grade transport is not a v0 implementation requirement. |
| Raw socket transport, host-to-host networking, distributed federation, WAN RPC | Out of v0 | v0 proves local grant semantics first (`vision.md` "Project aim"). |
| Application-level request/response RPC protocols | Out of v0 | v0 is stream-first. Grant negotiation uses request/reply internally; arbitrary RPC protocols can wait. **Confidence: medium - this is the strongest scope cut and should be revisited after `architecture/concepts.md`.** |
| Dynamic protocol/plugin loading, trusted module loading, runtime extension ABI | Out of v0 | v0 can hand-register protocol definitions; dynamic loading is not needed to prove the algebra (`vision.md` "Protocol/plugin extensibility"). |
| Durable grants, persistent capabilities, cross-run restoration | Out of v0 | v0 grants are runtime-scoped and revocation-aware; persistence adds a separate identity and storage problem. |
| Cryptographic capability format or OS-handle capability format | Out of v0 decision; open for ADR | v0 requires unforgeable-by-API grant handles conceptually, not a final deployment threat model (`tenets.md` Tenet 3). |
| Global topic strings as authority | Out of v0 | A protocol name may identify a protocol, but authority to use it comes only from a grant (`tenets.md` Tenet 3). |
| Full compiler/codegen toolchain | Out of v0 | Hand-authored protocol declarations are enough for the first implementation; generated bindings are v1+ candidate work. |

## In-scope-for-v0

| Concern | Spec v0 | First implementation v0 | Portability path | Why it matters |
|---|---|---|---|---|
| Protocol identity and typed payloads | Protocols have stable identifiers, typed payload declarations, stream kind, rights vocabulary, and compatibility metadata. | Protocols can be hand-written C++ types and descriptors with compile-time layout checks where eligible. | Protocols can map to generated or hand-written static types in other systems languages. | Typing is the entry point for unifying policy with communication (`vision.md` "First-class typed protocols"). |
| Grant record | A grant carries protocol, rights, tenant, persona, mode, delivery policy, resource limits, lifecycle state, and epoch. | A `Grant` object is required for publisher/subscriber construction and runtime calls. | A grant can become an opaque handle plus metadata table entry. | The grant is the bearer of authority (`tenets.md` Tenet 3). |
| Epoch semantics | An epoch is a per-grant monotonic counter. Sends and receives carry the epoch. Stale-epoch sends are rejected. Revocation propagation and in-flight sample semantics remain open questions. | Runtime stores epochs in the grant table, stamps send/receive operations, and raises a named stale-epoch error for stale sends. | Other implementations can preserve the same table and error semantics. | Epochs make grant changes observable without making mode, revocation, or downgrade state ambient (`tenets.md` Tenet 3). |
| Grant lifecycle | v0 names at least `requested`, `approved`, `denied`, `active`, `suspended`, `revoked`, and `expired`. | Runtime methods must surface these states and reject invalid transitions. | Lifecycle checks become cheap table/state checks. | Revocation and downgrades must be observable (`vision.md` "Revocation-aware design"). |
| JIT authorization | Grant requests can be approved, denied, downgraded, mode-restricted, quota-restricted, time-limited, suspended, or revoked. | A policy callback or small authorizer object can decide requests. | A user-space authorizer service can produce the same grant records. | Grant issuance is where tenant/persona/mode composition becomes real (`vision.md` "JIT authorization"). |
| Tenant hierarchy | Tenants are hierarchical identifiers, not process IDs. A grant names exactly one effective tenant path. | Runtime models tenant paths as immutable values and can intern tenant IDs for quota accounting. | Other implementations preserve tenant paths as grant metadata. | Tenancy is a policy input and resource-accounting key (`vision.md` "Multi-tier multi-tenancy"). |
| Persona views | A persona selects a view of a protocol: allowed rights, visible fields or stream classes, and allowed delivery policies. | Runtime enforces view masks or narrowed wrappers at publisher/subscriber boundaries. | Generated or hand-written bindings can expose narrowed wrappers. | Persona is not an ACL label; it changes the protocol view (`vision.md` "Persona-aware access"). |
| Mode-bound delivery | A mode selects delivery behavior: visible, hidden, realtime, background, alert-only, or keep-warm as v0 named modes. | Runtime maps modes to queues, wait sets, timers, and wake hints. | Other implementations preserve the same observable delivery semantics. | Mode must affect observable delivery, not documentation only (`vision.md` "Mode-aware delivery"). |
| Pub/sub fanout | A publisher with a write grant can publish to multiple subscribers with compatible read grants. Each subscriber's grant controls its delivery view. | Runtime can fan out references or copied samples behind the same `Sample` API. | Future transports can replace the backing store without changing grant semantics. | Fanout is central to the desktop/plugin/service-graph use case (`vision.md` "Publish/subscribe and fanout support"). |
| Latest-state blackboard | A state protocol can replace history with current value, and late subscribers can receive the latest grant-visible state. | Runtime stores current state per protocol/tenant/mode partition with epoch/version checks. | Other implementations can map this to shared state cells. | This proves GrantNet is not only an event bus (`vision.md` "Blackboard/latest-state model"). |
| Backpressure and overflow | v0 includes `blocking`, `bounded_history`, and `latest_only`. `lossy`, `batched`, and richer combinations are deferred unless needed by a demonstration path. | Runtime enforces queue depth/history/latest replacement behavior. | Other implementations preserve the same policy names and observable outcomes. | Delivery policy must be a grant-visible contract (`vision.md` "Backpressure and overflow policy"). |
| Wake behavior | v0 includes wake hints as delivery semantics: wake per item, wake on batch, wake on latest replacement, or suppress while hidden. | Runtime maps hints to event loops, wait sets, or notification objects. | Other implementations preserve the same wake contract. | Wake behavior is part of protocol policy, not an OS optimization only (`vision.md` "Integrated wake/sleep behavior"). |
| Resource limits | Grants can carry queue depth, history size, payload size, wake budget, and optional expiration. | Runtime checks limits before enqueue/deliver and accounts them against grant/tenant state. | Other implementations can enforce limits in memory pools and schedulers. | Resource governance becomes grant-local instead of ambient (`vision.md` "Resource governance"). |
| Revocation | Revoked grants reject future grant use. Revocation propagation and in-flight sample behavior are open questions. | Runtime marks grant-table entries revoked and raises a named error for future use. | Other implementations preserve the same rejection semantics. | Revocation distinguishes grants from plain subscriptions (`vision.md` "Revocation-aware design"). |
| Developer-facing API shape | v0 names core API concepts: `Runtime`, `Protocol`, `GrantRequest`, `Grant`, `Publisher`, `Subscriber`, `Sample`, `StateCell`, and named grant errors. | First implementation uses these names unless a language-specific spelling ADR overrides them. | Future SDKs reuse names and error semantics. | The spec must be recognizable across implementations (`tenets.md` Tenet 2; `tenets.md` Tenet 6). |
| Zero-copy eligibility | The spec distinguishes owned payloads, borrowed samples, release/ack behavior, and payloads eligible for fixed-layout shared-memory transport. | Runtime returns `Sample` objects with explicit release/ack behavior even when the backing store is simple. | Future transports can loan chunks and reference-count or lease them. | This keeps the future iceoryx-like path open without letting performance drive v0 (`tenets.md` Tenet 6). |

## Deferred-to-v1

1. **Actual user-space zero-copy transport.** A v1 systems-language runtime should prove shared-memory pools, loaned chunks, release semantics, and fanout without copies. It remains user-space (`tenets.md` Tenet 5).

2. **Formal protocol schema and code generation.** v1 should decide whether GrantNet uses a bespoke schema, adapts an existing IDL, or keeps a library-first declaration model. v0 may hand-author protocols.

3. **Dynamic discovery and grant-routed registry.** v1 should add a registry that can discover protocol providers without making names into authority. The registry itself must be grant-routed (`tenets.md` Tenet 3).

4. **Dynamic protocol/plugin modules.** v1 can load trusted protocol families or extension modules after the grant algebra is stable enough to police them.

5. **Application-level RPC.** v1 can add request/response protocols if they compose cleanly with grants. v0 should not inherit RPC-first assumptions from Binder, FIDL, or Cap'n Proto.

6. **Delegation and attenuation.** v1 should decide whether a grant holder can derive a narrower grant for another tenant/persona/mode, and how provenance is represented.

7. **Persistent grants.** v1+ can consider save/restore semantics after the runtime-scoped grant model is stable.

8. **Network transport and gateways.** DDS, Zenoh, gRPC, or custom network bridges are v1+ or later. They must preserve grant semantics rather than tunneling raw topics.

9. **Richer backpressure taxonomy.** `lossy`, `batched`, priority-aware, deadline-aware, and multi-policy combinations are deferred unless v0 examples prove they are necessary.

10. **OS scheduler and power integration.** v1 can map wake hints to platform primitives. v0 only specifies and simulates visible behavior.

11. **Threat model and capability representation ADRs.** v1 should resolve token vs OS handle vs cryptographic capability after v0 proves the conceptual model.

12. **Diagrams, versioning policy, and governance.** These remain project open questions, not v0 scope (`PLAN.md` "Open questions").

## Open questions

1. **Grant representation.** Is the future systems-language grant best represented as an opaque integer handle, an OS handle, a cryptographic token, or a structured object that indexes runtime state? This is unresolved and belongs in the M3 capability-model ADR (`PLAN.md` M3).

2. **Composition order.** Does grant issuance evaluate tenant first, then persona, then mode, then protocol policy; or does protocol policy define the matching order? The v0 scope requires composition, but `architecture/concepts.md` must settle the algebra (`PLAN.md` M2).

3. **Revocation propagation and in-flight samples.** When a grant is revoked, how does revocation propagate to publishers, subscribers, queued deliveries, and borrowed samples? May already-received samples remain readable until released, or must they become invalid immediately? The first implementation may choose a temporary behavior, but the spec must decide before a zero-copy runtime exists.

4. **Mode change trigger.** Does the runtime observe mode changes as explicit grant rebind requests, runtime events, or tenant-context transitions? Ambient mutation is out; the trigger shape remains unresolved.

5. **Mode change grant mechanic.** What's the precise mechanic by which a mode change is observable through grants - new epoch, suspend plus reissue, or revalidation? v0 must commit to one before any subscriber's mode can change.

6. **Schema surface.** Does v0 use a declarative mini-IDL, C++-first protocol descriptors, or a Markdown spec plus hand-authored code? The scope allows hand-authored protocols; the authoring surface remains open.

7. **First implementation language.** C++ is the current candidate because it can exercise layout, ownership, and a later kernel-adjacent path in one language family. The language choice still needs an ADR before implementation work hardens around local idioms. **Confidence: medium - C++ fits the path, but this repository has not yet recorded the decision.**

8. **Process boundary.** Should v0 demonstrate separate OS processes, cooperative tasks in one process, or both? The scope requires the API and behavior; the concrete topology should be decided before implementation begins.

9. **Default policies.** If a protocol omits delivery, backpressure, wake, or resource limits, should v0 reject the protocol or apply conservative defaults? This is unresolved because defaults can become ambient policy if handled carelessly (`tenets.md` Tenet 3).

10. **Grant-visible field views.** Persona-aware access may require payload field filtering. Whether v0 implements field-level views or only stream/right-level views remains unresolved.

## What v0 must demonstrate

v0 is real if the spec and first implementation can demonstrate these code paths with API names and error semantics that future implementations can keep (`tenets.md` Tenet 2; `tenets.md` Tenet 6).

1. **Typed fanout with grant-filtered subscribers.**
   * Define one typed event protocol, such as `WorkspaceEvent`.
   * Issue a write grant to a publisher tenant.
   * Issue two read grants to different subscriber tenants/personas.
   * Publish one event.
   * Show each subscriber receives only the view and delivery behavior authorized by its grant.
   * Proves: typed protocols, grants, tenants, personas, pub/sub fanout.

2. **Mode-aware delivery at grant issuance.**
   * Start one subscriber with a `visible`-mode grant and bounded history.
   * Start another subscriber with a `hidden`-mode grant and latest-only, suppressed, or batched delivery.
   * Publish several events.
   * Show visible mode receives normally and hidden mode receives according to the hidden-mode grant.
   * Proves: mode is grant-visible, not ambient runtime state.

3. **Latest-state blackboard.**
   * Define one typed state protocol, such as `ThemeState` or `PowerState`.
   * Publish multiple state replacements.
   * Add a late subscriber with a compatible grant.
   * Show it receives the latest grant-visible state, not the whole history.
   * Proves: blackboard/latest-state and stream fanout share the same grant model.

4. **Revocation and stale-epoch send rejection.**
   * Issue an active write grant and an active read grant.
   * Deliver at least one sample.
   * Replace the write grant with a downgraded or newer epoch.
   * Show a send carrying the old epoch fails with a named stale-epoch error.
   * Revoke the read grant and show future receive/poll/open calls fail with a named grant error.
   * Proves: grants are live authority, not subscription receipts.

5. **Resource-limit enforcement.**
   * Issue a grant with queue depth or history size limits.
   * Publish beyond the limit under `blocking`, `bounded_history`, and `latest_only`.
   * Show each policy's observable result.
   * Proves: backpressure and overflow are grant contracts.

6. **No ambient authority path.**
   * Attempt to publish, subscribe, or read latest state without a grant.
   * Show the runtime rejects the action before transport or topic lookup.
   * Proves: GrantNet has not split authority between grants and side channels (`tenets.md` Tenet 3).

## Prior art shaping v0

| Prior art | v0 decision it shapes | GrantNet divergence |
|---|---|---|
| Fuchsia FIDL over Zircon channels | v0 adopts the separation between protocol definition, bindings, and underlying transport. FIDL defines protocols and generates language bindings, while Fuchsia notes that FIDL bindings use channel communication rather than the OS knowing FIDL intrinsically. Zircon channels also show handle transfer as an IPC primitive. | GrantNet puts tenant/persona/mode/resource/wake semantics into the grant and protocol contract, not merely into an IDL plus channel handle. v0 is not tied to a kernel channel transport. Sources: [FIDL overview](https://fuchsia.dev/fuchsia-src/concepts/fidl/overview), [Zircon fundamentals](https://fuchsia.dev/fuchsia-src/get-started/learn/intro/zircon). |
| iceoryx | v0 preserves a future loaned-sample API and bounded queues because iceoryx demonstrates publisher-written shared-memory chunks and subscriber references as an end-to-end zero-copy model. | GrantNet does not make true zero-copy throughput an acceptance gate for v0. GrantNet also rejects service-description strings as authority; a matching protocol name is not enough without a grant. Sources: [iceoryx overview](https://iceoryx.io/v2.0.5/getting-started/overview/), [What is Eclipse iceoryx?](https://iceoryx.io/v1.0.1/getting-started/what-is-iceoryx/). |
| Cap'n Proto RPC | v0 takes capability discipline seriously: a reference should both designate and authorize. Cap'n Proto also warns that protocol complexity can preserve clean object structure and avoid ad-hoc latency workarounds. | GrantNet v0 is not distributed-object RPC. Grants authorize typed streams and state cells with policy metadata. Persistent capabilities, three-way interactions, and reference equality are deferred. Source: [Cap'n Proto RPC Protocol](https://capnproto.org/rpc.html). |
| DDS | v0 includes data-centric pub/sub, matching producers to consumers, and delivery semantics as first-class concerns. DDS frames this as APIs plus communication semantics/QoS for delivering the right information to matching consumers. | GrantNet does not adopt DDS topics, global discovery, QoS profiles, or wire compatibility in v0. Delivery semantics live inside grant-routed protocol use, not alongside it as an external QoS layer. Source: [OMG DDS 1.4](https://www.omg.org/spec/DDS/). |
| seL4 | v0 follows the capability-centered lesson: authority is held in capabilities, IPC can transfer capabilities, and badges can distinguish senders through capability metadata. seL4 notifications also show why wake/signal behavior should be explicit. | GrantNet v0 is user-space and does not model CSpaces, kernel endpoints, formal verification, or seL4 fastpath constraints. It borrows capability discipline, not the kernel API. Sources: [seL4 capabilities tutorial](https://docs.sel4.systems/Tutorials/capabilities.html), [seL4 IPC tutorial](https://docs.sel4.systems/Tutorials/ipc), [seL4 notifications tutorial](https://docs.sel4.systems/Tutorials/notifications.html). |
| Android Binder | v0 adopts the ergonomics lesson that proxy-like typed calls are approachable, and the caution that kernel drivers can hide copies, scheduling, and service-manager authority behind a friendly interface. Binder's domain separation also shows that IPC namespace boundaries matter. | GrantNet v0 rejects Binder's kernel-driver dependency, service-manager lookup as authority, and transparent remote-call illusion. Grant use and grant failure stay explicit. Sources: [Binder overview](https://source.android.com/docs/core/architecture/ipc/binder-overview?hl=en), [Use Binder IPC](https://source.android.com/docs/core/architecture/hidl/binder-ipc?hl=en). |

## References

* `tenets.md` - binding design constraints, especially Tenet 1, Tenet 3, Tenet 5, and Tenet 6.
* `vision.md` - founding GrantNet concerns and one-sentence description.
* `PLAN.md` - M1 deliverable shape, M2/M3 follow-on milestones, and project open questions.
* `DESIGN_NOTES.md` - citation discipline, provider abstraction, and scope-wall reminders.
* Fuchsia FIDL overview - <https://fuchsia.dev/fuchsia-src/concepts/fidl/overview>
* Fuchsia Zircon fundamentals - <https://fuchsia.dev/fuchsia-src/get-started/learn/intro/zircon>
* iceoryx overview - <https://iceoryx.io/v2.0.5/getting-started/overview/>
* iceoryx concept introduction - <https://iceoryx.io/v1.0.1/getting-started/what-is-iceoryx/>
* Cap'n Proto RPC protocol - <https://capnproto.org/rpc.html>
* OMG DDS 1.4 - <https://www.omg.org/spec/DDS/>
* seL4 capabilities tutorial - <https://docs.sel4.systems/Tutorials/capabilities.html>
* seL4 IPC tutorial - <https://docs.sel4.systems/Tutorials/ipc>
* seL4 notifications tutorial - <https://docs.sel4.systems/Tutorials/notifications.html>
* Android Binder overview - <https://source.android.com/docs/core/architecture/ipc/binder-overview?hl=en>
* Android Binder IPC notes - <https://source.android.com/docs/core/architecture/hidl/binder-ipc?hl=en>

Confidence: medium - v0 no longer assumes a preliminary demo language and C++ is recorded as the current first-implementation candidate; confidence is not high until a language-choice ADR lands.
