# GrantNet — Vision

The founding vision summary, captured at project inception (2026-05-07). Authoritative on intent and scope ambition; not a substitute for resolved spec. As `architecture/SCOPE.md` and `architecture/spec/` mature, they bind tighter than this document for any specific question — this document remains as the "why" that shaped them.

---

## Project aim

**GrantNet** is an experimental IPC/network substrate for building high-performance, modular systems where communication is not opened through raw sockets or loose topic strings, but through **typed, tenant-scoped, persona/mode-aware grants**.

The goal is to make protocols themselves first-class system components: statically typed, permission-aware, mode-aware, and optimized for high-performance zero-copy local streaming.

## Value proposition

Modern IPC systems usually separate transport, schemas, authorization, tenancy, scheduling, wake behavior, and plugin lifecycle into different layers. GrantNet aims to unify those concerns at the protocol level.

Instead of asking:

> "Can this process connect to this socket?"

GrantNet asks:

> "Should this tenant, acting under this persona, in this mode, receive this typed stream with these permissions, performance guarantees, and resource limits?"

This enables a safer and more efficient foundation for desktop environments, plugin systems, local service graphs, driver-adjacent runtimes, and eventually kernel-integrated communication paths.

## Core features

* **First-class typed protocols**
  * Protocols define payload layout, stream behavior, permissions, tenancy rules, wake behavior, and compatibility expectations.
  * Protocols can be compiled into the runtime, loaded as trusted modules, or generated from schema definitions.

* **Grant-bound communication**
  * Access to a stream is granted explicitly through a capability-like handle.
  * A grant carries the protocol type, rights, tenant identity, active mode, resource limits, and revocation state.

* **JIT authorization**
  * Access decisions are made at the moment a plugin or process requests a stream.
  * Grants can be approved, denied, downgraded, time-limited, revoked, or mode-restricted.

* **Multi-tier multi-tenancy**
  * Tenants can represent users, sessions, applications, plugins, windows, services, sandboxes, or system domains.
  * Each tenant can have separate visibility, quotas, permissions, memory regions, and stream access.

* **Persona-aware access**
  * A client can request a stream under a specific persona, such as widget, scientist, admin tool, compositor plugin, background service, or debug inspector.
  * Persona determines the available view of a protocol, not just whether access is allowed.

* **Mode-aware delivery**
  * Streams can adapt based on runtime mode, such as visible, hidden, realtime, decimated, background, alert-only, or keep-warm.
  * Mode can affect delivery rate, wake frequency, history retention, caching behavior, and priority.

* **High-performance zero-copy local streams**
  * Designed for same-system IPC using shared-memory-style data paths.
  * Hot-path data can be transferred through fixed-layout typed payloads without serialization or redundant copying.

* **Socket-like developer ergonomics**
  * Exposes familiar stream-oriented communication patterns while preserving stronger typing, grants, tenancy, and policy semantics.

* **Publish/subscribe and fanout support**
  * Supports one-to-many typed streams where multiple subscribers can receive the same published data without requiring the publisher to manually duplicate payloads.

* **Integrated wake/sleep behavior**
  * Stream contracts can define when subscribers should wake, sleep, batch, skip, or receive deadline notifications.
  * Useful for lightweight desktop environments where hidden or idle components should not waste CPU.

* **Backpressure and overflow policy**
  * Protocols can define whether streams are lossless, lossy, latest-only, bounded-history, batched, or blocking.
  * Different stream classes can use different delivery policies.

* **Blackboard/latest-state model**
  * Supports state-oriented protocols where consumers need the latest value rather than every historical event.
  * Useful for configuration, theme state, workspace state, power state, and similar data.

* **Revocation-aware design**
  * Grants can carry epochs, expiration, and revocation metadata.
  * The runtime can prevent future access after a permission change or tenant downgrade.

* **Resource governance**
  * Grants can include quotas for memory, wake frequency, queue depth, history size, bandwidth, and CPU-sensitive behaviors.
  * Prevents plugins or tenants from accidentally or maliciously overwhelming the system.

* **Compile-time language support**
  * Intended for high-performance SDKs in systems languages such as Rust, C, C++, Zig, or similar compile-time languages.
  * Hot-path protocol wrappers can be statically generated and optimized.

* **Protocol/plugin extensibility**
  * New protocol families can be added as first-class modules.
  * Trusted protocols may be compiled directly into the runtime for maximum performance.

* **Kernel/driver integration path**
  * Long-term goal is to support lower-level integration where grants, memory regions, wakeups, and typed protocols can interact with custom drivers or kernel-adjacent components.
  * Out of scope for v0 and v1; expected to be carried by the open-source community as the project matures (see `tenets.md` § Tenet 5).

## One-sentence description

**GrantNet is a typed capability IPC substrate for tenant-scoped, persona/mode-aware, JIT-authorized, high-performance zero-copy local streams.**
