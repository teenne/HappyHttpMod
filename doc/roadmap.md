# Roadmap

This roadmap describes where the project is going and how we decide what to work on next.

It is intentionally lightweight. The source of truth for supported targets is [docs/versions.md](versions.md). Work is tracked in GitHub issues.

## Goals

- Support multiple Minecraft versions without maintaining separate repositories
- Support multiple loaders (Forge, NeoForge, Fabric) without duplicating core logic
- Keep the mod stable on supported versions while making it easy to port forward
- Make builds reproducible and easy for contributors (one command builds all jars)

## Current state

- Baseline implementation: **Forge 1.20.2**
- Many compatibility requests exist as separate issues. We are consolidating these into:
  - a version support matrix in `docs/versions.md`
  - one compatibility tracker issue per loader

## Near-term priorities

### 1) Multi-module build and repo structure

We are migrating towards a structure with one module per loader and Minecraft version.

Deliverables
- Clear module boundaries (`common` vs version modules)
- Root tasks to build all modules and collect jars
- Jar naming includes loader and Minecraft version

### 2) Forge support for newer Minecraft versions

Primary target
- Forge 1.21.x (pin a specific version as the first milestone, then expand)

Deliverables
- A new Forge version module builds and launches
- Both blocks pass the minimum smoke test
- `docs/versions.md` updated with status

### 3) Reduce setup friction and improve usability

We will prioritise small changes that reduce user mistakes and support burden, for example:
- clearer configuration feedback
- safer defaults
- better diagnostics

These should be designed so they can live in `common` where possible, and implemented with minimal version-specific glue.

## Mid-term priorities

### NeoForge support

Once Forge is stable on the new version line, we will evaluate NeoForge support for the same Minecraft version range.

### Fabric support

Fabric support will follow after:
- the multi-module build is stable
- Forge ports are in a repeatable state
- we can maintain compatibility without duplicating core logic

## How we decide what is next

We prioritise work that:

1. Unlocks multi-version support without breaking the baseline
2. Reduces ongoing maintenance cost
3. Improves reliability and observability (better errors, logs, and diagnostics)
4. Has clear user value and can be tested

We de-prioritise work that:
- is large but vague
- adds significant complexity for every target
- cannot be reasonably tested across the supported matrix

## Release policy (initial)

- We release per target module (loader + Minecraft version)
- Every release should clearly state:
  - supported loader
  - supported Minecraft version
  - notable changes and known issues

Release notes should link to the relevant issues and PRs.

## How to contribute

- Work is tracked as GitHub issues: https://github.com/Narratimo/HappyHttpMod/issues
- Contribution workflow: [CONTRIBUTING.md](../CONTRIBUTING.md)
- Porting steps: [docs/porting-guide.md](porting-guide.md)
- Repo layout rules: [docs/repo-layout.md](repo-layout.md)

If you want to help with ports, start by:
- checking [docs/versions.md](versions.md)
- commenting in the compatibility tracker issue for the loader you want to work on
- picking one target module and keeping the PR small
