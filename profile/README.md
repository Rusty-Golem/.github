# 🗿 Rusty Golem

A clean-room, modular **Rust** suite for **Minecraft: Java Edition** (target **1.21.x**) — a stack of
small, fast, headless crates that compose into a bot, with independent systems (surveillance, live
feed, AI control, MCP) built on top. The UI is always a separate project.

> **Clean-room, never guessed.** Behavior comes from public protocol documentation
> (minecraft.wiki / wiki.vg) and the official Mojang data-generator output — codegenned at build
> time, not hand-transcribed and not copied from other implementations. Every crate is validated
> against an authoritative artifact (a real `level.dat`, a real region file, a live server).

## The suite at a glance

```
systems     watch · feed · harness · mind · mcp          (each depends only on bot-core)
   ▲
bot-core    connect · world-state · control · events     (the stable contract)
   ▲
foundation  golem-wire · golem-nbt · golem-data · golem-types · golem-proto ·
            golem-auth · golem-world · golem-physics · golem-chat
```

You can't build a tier before the one below it. Each crate is its own repository with full docs,
tests, and CI (`fmt` + `clippy -D warnings` + `test`).

## Foundation crates

| Crate | What it does |
|-------|--------------|
| [`golem-wire`](https://github.com/Rusty-Golem/golem-wire) | The lowest layer: var-int/var-long and primitive read/write. Zero-copy decode, no panics on malformed input, std-only. |
| [`golem-nbt`](https://github.com/Rusty-Golem/golem-nbt) | A clean-room NBT (Named Binary Tag) codec — all 13 tag types, Java Modified UTF-8, both disk and network framings. |
| [`golem-data`](https://github.com/Rusty-Golem/golem-data) | Feature-gated Minecraft data tables generated at build time from the official Mojang data-generator output. One module per domain (blocks, items, registries, recipes, tags, …); consumers compile only what they enable. |
| [`golem-types`](https://github.com/Rusty-Golem/golem-types) | Core domain value types shared everywhere (`BlockPos`, `Vec3`, `GameProfile`, `ItemStack`, …), each carrying its own wire encode/decode. The single source of truth with no circular dependencies. |
| [`golem-proto`](https://github.com/Rusty-Golem/golem-proto) | The protocol layer: framing, handshake, status, and the full login → configuration → play path, with typed structs for the 1.21.1 packet set. Every packet round-trip tested. |
| [`golem-auth`](https://github.com/Rusty-Golem/golem-auth) | Authentication primitives: offline-mode UUIDs, the online-mode encryption handshake (RSA + AES-128-CFB8), and Microsoft/Xbox device-code login. |
| [`golem-world`](https://github.com/Rusty-Golem/golem-world) | Paletted chunk storage plus both Anvil region-file (on-disk) and network (over-the-wire) chunk parsing, sharing one validated decode path. |
| [`golem-physics`](https://github.com/Rusty-Golem/golem-physics) | Headless player-movement physics: vanilla gravity/drag integration and AABB collision against a world chunk. Constants cited, never guessed. |
| [`golem-chat`](https://github.com/Rusty-Golem/golem-chat) | Minecraft text components — the rich-text model for chat, MOTDs, and item names. Parse/serialize JSON, flatten to plain text, legacy `§`-code and NBT support. |

## `bot-core` — the one thing everything needs

[`bot-core`](https://github.com/Rusty-Golem/bot-core) composes the foundation into one ergonomic,
**stable, headless contract**:

> **connect → reach Play → observe → act**

A `Bot` owns a connection that has already traversed handshake → login → configuration → play. You
drive it by pumping events (`Spawned`, `Chat`, `Moved`, `BlockChanged`, `EntitySpawned`,
`InventoryUpdated`, …); keep-alives are answered internally. The public surface is deliberately tiny
and fully documented so it can stay stable as the suite grows — every higher-level system codes
against *this* crate, never the protocol internals beneath it.

## Systems (built on `bot-core`)

Each is an independent repository that depends only on `bot-core`:

- **`watch`** — surveillance: observe and record world/player events, motion/diff detection, alerts.
- **`feed`** — live point-of-view stream (renderer + WebSocket/WebRTC out).
- **`harness`** — orchestrate fleets of bots via external harnesses and schedulers.
- **`mind`** — full AI/LLM perception → plan → act control.
- **`mcp`** — wrap `bot-core` as an MCP server (connect/observe/act exposed as tools).

The UI is always a separate project that talks to systems over IPC/HTTP/WS — it never depends on the
core directly.

## `forge`

[`forge`](https://github.com/Rusty-Golem/forge) is the org's workshop: design tenets, architecture,
the dependency-ordered roadmap, per-crate plans, and the public website. **Start there** to
understand what's being built and why.

## Principles

Clean-room · small reusable crates · minimal dependencies · headless-first · fully tested (TDD + CI)
· well documented · optimized and measured · codegen from data · stable contracts.

## Status

Phase 1 complete: the full clean-room foundation plus `bot-core` is built — every crate 1.21.1-aligned
and validated against an authoritative artifact. The clean-room Rust stack connects, reaches Play, and
chats on a real 1.21.1 server. Phase 2 is the systems layer. See the
[roadmap](https://github.com/Rusty-Golem/forge/blob/main/docs/roadmap.md) for live status.

## License

Dual-licensed under MIT or Apache-2.0.
