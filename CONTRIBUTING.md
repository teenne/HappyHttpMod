# Contributing to the Happy HTTP Minecraft mod

Thanks for contributing. This project is an open source Minecraft mod that aims to support multiple Minecraft versions and multiple loaders (Forge, NeoForge, Fabric) from one repository, while keeping the codebase easy to work on and safe to change.

This guide explains how we coordinate work, where to put code, and how to submit changes without breaking other versions.

## Table of contents

1. [Code of conduct](#code-of-conduct)
2. [Where decisions and work are tracked](#where-decisions-and-work-are-tracked)
3. [Repository layout and responsibilities](#repository-layout-and-responsibilities)
4. [How to contribute](#how-to-contribute)
5. [Reporting bugs](#reporting-bugs)
6. [Suggesting features](#suggesting-features)
7. [Contributing code](#contributing-code)
8. [Development setup and build commands](#development-setup-and-build-commands)
9. [Coding standards](#coding-standards)
10. [Pull request process](#pull-request-process)
11. [Issue templates and labels](#issue-templates-and-labels)
12. [Community and support](#community-and-support)

## Code of conduct

This project and everyone participating in it is governed by the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behaviour as described in that document.

## Where decisions and work are tracked

We use two channels:

- Discord is for quick coordination, questions, and helping contributors get unblocked
- GitHub is the source of truth for decisions, work items, and changes

Rules

- If it affects the codebase, it must be captured in GitHub as an issue, pull request, or documentation change
- Decisions should be written into the relevant GitHub issue or pull request, even if they were discussed on Discord
- Please avoid “design by Discord message”. Link to an issue, propose the change there, then implement via a PR

GitHub Issues: https://github.com/Narratimo/HappyHttpMod/issues

## Repository layout and responsibilities

We are moving towards a multi-module Gradle build where each produced jar corresponds to a specific loader and Minecraft version.

This keeps the mod stable for existing versions while allowing development on newer versions in parallel.

Target structure (illustrative)

```
common/                         Shared logic used by all loaders and versions

forge/
  mc_1_20_2/                    Forge implementation for Minecraft 1.20.2
  mc_1_20_4/                    Forge implementation for Minecraft 1.20.4
  mc_1_21_1/                    Forge implementation for Minecraft 1.21.1

fabric/
  mc_1_20_2/
  mc_1_21_1/

neoforge/
  mc_1_21_1/
```

What goes where

- `common/`
  - Gameplay logic, algorithms, configuration models, shared utilities
  - Code that should behave identically for every supported version/loader
  - Shared assets and data where formats allow it
- `forge/mc_*`, `fabric/mc_*`, `neoforge/mc_*`
  - Loader entrypoints and mod bootstrap
  - Registration code and event wiring
  - Mixins that target loader/version-specific classes and signatures
  - Version-specific resources and metadata (for example `META-INF/mods.toml`)

Rule of thumb

- If the code touches Minecraft or loader APIs and it differs between versions, it belongs in a version module
- If the logic should be identical across modules, it belongs in `common`

Build outputs

- Each module produces jars in its own `build/libs/`
- The root build will also provide a task that collects all jars into a single folder

Jar naming policy

- Jars must include both loader and Minecraft version so they never overwrite
  - Example: `happyhttp-1.2.3-forge-mc1.20.2.jar`

## How to contribute

Ways to help

- Report bugs with logs and reproduction steps
- Suggest features with a clear use case
- Fix bugs (especially in a specific module)
- Port the mod to a newer Minecraft version under one loader
- Reduce duplication by moving shared logic into `common`
- Improve docs, build scripts, CI, and contributor workflow

Before starting a larger change

- Check if there is an existing issue
- If there isn’t, create one describing what you intend to do
- If you are unsure about the approach, ask in Discord and link the issue for context

## Reporting bugs

Please report bugs using GitHub Issues:
https://github.com/Narratimo/HappyHttpMod/issues

When reporting a bug, include

- Minecraft version
- Loader (Forge, Fabric, NeoForge)
- Mod version
- Client or server (or both)
- Steps to reproduce
- Expected vs actual behaviour
- Logs and crash reports

Logs and crash reports

- Prefer attaching `latest.log` and any crash report file
- If you paste logs, use a fenced code block and remove secrets (tokens, IPs)

## Suggesting features

We welcome feature suggestions via a GitHub issue.

Include

- Problem and use case (who needs this and why)
- Proposed behaviour
- Any compatibility concerns (client/server, required side, version constraints)
- Alternatives you considered

Please avoid implementing large features without an issue first. We want to ensure feature direction aligns with multi-version + multi-loader goals.

## Contributing code

### 1. Fork and clone

Fork the repository and clone your fork.

```
git clone https://github.com/<your-username>/HappyHttpMod.git
cd HappyHttpMod
```

### 2. Create a branch

Use a descriptive branch name.

```
git checkout -b fix/short-description
```

Examples

- `fix/forge-1_20_2-timeout`
- `port/forge-1_21_1`
- `refactor/common-http-core`
- `build/multi-module-tasks`

### 3. Make changes in the correct module

When possible

- Implement shared logic in `common/`
- Keep loader/version modules as thin glue layers
- Avoid copying code between modules unless unavoidable, and explain why in the PR

### 4. Commit messages

Use clear, descriptive commits.

Examples

- `Fix: null pointer in request parsing`
- `Port: Forge 1.21.1 build`
- `Refactor: extract shared HTTP core into common`

### 5. Push and open a pull request

```
git push origin fix/short-description
```

Open a PR to the main repository and fill in the template.

## Development setup and build commands

Prerequisites

- JDK (use the version required by the target Minecraft/loader module)
- Git
- A working Gradle wrapper (`./gradlew`)

Build commands

- Build everything (all supported modules)

```
./gradlew buildAll
```

- Build jars and collect them into one folder

```
./gradlew collectJars
```

- Build a single module (examples)

```
./gradlew :forge:mc_1_20_2:build
./gradlew :forge:mc_1_21_1:build
```

If the module path differs in this repo, use Gradle’s task listing to find the correct names:

```
./gradlew projects
```

Local verification expectations

- If you change only one loader/version module, build that module
- If you change `common/`, build all modules (`buildAll`)
- If you change Gradle wiring or root tasks, build all modules

## Coding standards

Keep the codebase readable and reviewable.

- Use consistent formatting and naming
- Keep changes focused and minimise unrelated edits
- Avoid large “format-only” changes mixed with logic changes
- Document public classes and any non-obvious behaviour
- Prefer composition and small interfaces between `common` and platform modules

Compatibility guidelines

- Do not use Minecraft internals in `common/`
- Keep mixin targets and signatures confined to the version module
- When adding a new feature, consider how it impacts all supported modules

## Pull request process

A PR is mergeable when

- It has a clear title and description
- It links to an issue (or explains why no issue is needed)
- It passes CI
- It follows repository layout rules
- It includes any necessary documentation updates

PR scope rules

- One change per PR
- Avoid combining refactors and features
- Avoid combining multi-version port work with unrelated improvements

Review expectations

- Changes that affect Gradle structure or root build logic should be reviewed by someone who owns build configuration
- Changes that affect a specific loader/version module should be reviewed by someone familiar with that module when possible

## Issue templates and labels

Please use issue templates if they are available in the repository.

Recommended labels

- `bug`
- `feature`
- `enhancement`
- `support`
- `performance`
- `refactor`
- `build`
- `port`
- `forge`
- `fabric`
- `neoforge`
- `mc-1.20.2`, `mc-1.20.4`, `mc-1.21.x` (or similar)

## Community and support

Discord

- Use Discord for quick questions and coordination
- If the question results in a change request, please create or link a GitHub issue

Be kind and constructive

- Assume good intent
- Help newcomers get unblocked
- Keep technical discussions grounded and specific

Thank you for helping improve Happy HTTP.
