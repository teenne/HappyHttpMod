# docs/architecture.md

This document explains the technical architecture of Happy HTTP, with a focus on how we keep the mod maintainable across multiple Minecraft versions and multiple loaders.

It is written to support a repo structure where each jar is produced by one module (loader + Minecraft version), with shared logic in `common/`.

## Goals

- Keep core behaviour consistent across supported targets
- Minimise duplicated code across loaders and Minecraft versions
- Isolate version-specific breakpoints to version modules
- Make ports predictable and reviewable

## High-level model

Happy HTTP is split into two layers:

1. Core logic (shared)
2. Platform glue (loader + Minecraft version)

### Core logic (shared)

The shared layer contains logic that should not care which loader or Minecraft version is running.

Examples
- HTTP request and response handling logic (as far as possible)
- Parsing and validation of configured endpoints and parameters
- Data models for configuration and block state
- Internal domain concepts, such as:
  - Receiver configuration
  - Sender configuration
  - Result outcomes and error states

This code lives in `common/`.

### Platform glue (version modules)

Each version module adapts the shared logic to the platform.

Examples
- Mod entrypoint initialisation
- Registering blocks, items, and block entities
- Wiring events and ticking logic
- UI screen registration and networking hooks
- Mixins and targets that differ by version
- Loading and saving config in the correct lifecycle phase

This code lives in `forge/mc_*`, `fabric/mc_*`, `neoforge/mc_*`.

## Dependency direction

The dependency direction is intentionally one-way:

- Version modules depend on `common`
- `common` must not depend on version modules

This keeps `common` portable and prevents “leaking” Minecraft internals into shared logic.

## Bridge interfaces

When shared logic needs a platform capability, define a small interface in `common` and implement it in the version module.

Examples of bridge needs
- Logging
- Scheduling work on the main thread
- Access to a safe configuration path
- Side detection (client vs server)
- A way to publish a redstone pulse result into the world

Guidelines
- Keep interfaces narrow
- Avoid exposing Minecraft objects directly through the interface unless you can keep them isolated and stable
- Prefer passing primitive values or small data objects

## HTTP server model

Happy HTTP runs an integrated HTTP endpoint that can receive requests and trigger in-game effects.

Key constraints across versions
- The mod must start and stop the HTTP server in the right lifecycle window
- Binding to an IP and port should be explicit and predictable
- The server must not block the game thread
- Request handling must be bounded to avoid runaway memory or CPU use

Default safety direction
- Prefer safe defaults
- Make risky behaviour explicit (for example, exposing endpoints publicly)

## Block model

### HTTP Receiver block

Inputs
- HTTP requests to a configured endpoint
- Optional parameter matching

Outputs
- Redstone output (on/off or pulse)

Responsibilities split
- `common`: parse and validate receiver configuration, match request parameters, decide output state
- version module: convert “trigger output” into the correct in-game redstone behaviour for that version

### HTTP Sender block

Inputs
- Redstone input signal

Outputs
- HTTP request to configured target endpoint

Responsibilities split
- `common`: validate sender config, build request data, represent results
- version module: trigger the action on signal change and handle lifecycle rules

## Multi-version design rules

- Keep all Minecraft/loader API differences in version modules
- Keep shared logic stable and testable in isolation
- If you must duplicate code across modules, document why
- Ports should be mostly mechanical changes to glue code, not re-architectures

## Observability direction (recommended)

To reduce support burden and speed up ports, we aim for:
- concise log messages with enough detail to diagnose
- per-block validation feedback in UI where possible
- optional debug mode to show request hits and sender outcomes

These should be implemented in a way that does not add heavy overhead or spam logs by default.

## Where to look next

- Repo structure: [docs/repo-layout.md](repo-layout.md)
- Porting steps: [docs/porting-guide.md](porting-guide.md)
- Support matrix: [docs/versions.md](versions.md)
- Contribution workflow: [CONTRIBUTING.md](../CONTRIBUTING.md)
