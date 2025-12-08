# Repository layout and module responsibilities

This repository is organised to support multiple Minecraft versions and multiple loaders from one codebase.

The goal is:

- One jar per loader and Minecraft version
- A shared core that avoids duplication
- A build that can produce all supported jars in one command

## The two axes

This repo has two axes:

1. **Loader**: Forge, NeoForge, Fabric  
2. **Minecraft version**: 1.20.2, 1.20.4, 1.21.x, and so on

Each supported combination of loader and Minecraft version is implemented as its own Gradle module.

## Target directory structure

Illustrative structure

```text
common/                         Shared logic used by all loaders and versions

forge/
  mc_1_20_2/                    Forge implementation for Minecraft 1.20.2
  mc_1_20_4/
  mc_1_21_1/

fabric/
  mc_1_20_2/
  mc_1_21_1/

neoforge/
  mc_1_21_1/
```

The exact set of modules will evolve. The source of truth for what is supported is [docs/versions.md](versions.md).

## What goes where

### common/

`common/` is for code that should behave identically across all supported targets.

Put code here when it is:
- Pure logic (parsing, validation, data structures, helpers)
- Configuration models that do not depend on Minecraft classes
- HTTP client/server logic that can be expressed without importing loader APIs
- Anything that would otherwise be duplicated across modules

Guideline
- Avoid importing Minecraft or loader classes into `common/`. If you need platform behaviour, define a small interface and implement it in the version module.

### Version modules (forge/mc_*, fabric/mc_*, neoforge/mc_*)

Each version module is responsible for:
- Loader entrypoints and initialisation
- Registration and event wiring
- Block and item registration and lifecycle hooks
- Mixins and their configs for that specific target
- Version-specific resources and metadata

Typical contents
- `src/main/java` with loader-specific and version-specific glue
- `src/main/resources`
  - Forge/NeoForge: `META-INF/mods.toml`
  - Fabric: `fabric.mod.json`
  - Mixin configs tied to that target

### Optional loader-common modules

As the module count grows, we may introduce loader-common modules to share glue that is common across multiple versions for the same loader.

Examples
- `forge/common/` or `forge/forge-common/`
- `fabric/common/`
- `neoforge/common/`

If these exist, they are allowed to depend on loader APIs but should avoid depending on Minecraft version-specific APIs as much as possible.

## Naming conventions

### Module folder names

- Loader folder: `forge`, `fabric`, `neoforge`
- Version module folder: `mc_1_20_2`, `mc_1_21_1`

Rules
- Use underscores in module folder names
- Do not use the word “latest” in folder names. Always pin the version.

### Jar naming

Jars must include both loader and Minecraft version so they never overwrite.

Example format

- `happyhttp-<mod_version>-forge-mc1.20.2.jar`
- `happyhttp-<mod_version>-forge-mc1.21.1.jar`

This is enforced by Gradle config in each version module.

## Build tasks

Expected tasks (may be added during refactor)

- Build everything
  - `./gradlew buildAll`

- Collect all jars into one folder
  - `./gradlew collectJars`

- Build one module (example)
  - `./gradlew :forge:mc_1_20_2:build`

If you are unsure about module task names, run:

```
./gradlew projects
```

## Rules that prevent drift

- Shared logic belongs in `common` unless it truly must be loader or version-specific
- Version modules should be thin and focused on integration and glue
- Do not duplicate logic across version modules without documenting why
- Keep ports small: one version module per PR is the default
- If you change `common`, you must ensure all modules still build

## Where to look next

- Supported versions matrix: [docs/versions.md](versions.md)
- Porting steps: [docs/porting-guide.md](porting-guide.md)
- Contribution workflow: [CONTRIBUTING.md](../CONTRIBUTING.md)
