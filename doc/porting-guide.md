# Porting guide: adding support for a new Minecraft version (Forge, NeoForge, Fabric)

This guide explains how to add a new Minecraft version target in this repository without breaking existing versions.

The goal is a repository where each produced jar corresponds to one loader and one Minecraft version, and we can build all supported jars in one command.

## Before you start

1. Check the current support matrix  
   See [docs/versions.md](versions.md).

2. Check if there is already a tracking issue  
   If a compatibility tracker exists for the loader you are working on, coordinate there instead of opening a new issue.

3. Pick one target at a time  
   Do not try to port multiple versions in one PR. One version module per PR is the default.

## Definitions

- **Common code**: shared logic that should behave the same across all versions and loaders.
- **Version module**: a Gradle module that builds exactly one jar for a specific loader and Minecraft version, for example `forge/mc_1_21_1`.

## What “done” means for a port

A version port is considered complete when:

- The module builds successfully
- The game launches (dev client)
- A world loads
- Both blocks can be placed and configured
- Receiver triggers redstone on request
- Sender sends a request on redstone input
- Config file generates correctly and settings take effect

If you can build but not run, mark the target as **Beta** in the version matrix and document what remains.

## Where changes should go

Put code in `common/` when:

- It is pure logic (parsing, validation, configuration models, utilities)
- It can be expressed behind a small interface without importing Minecraft or loader classes
- It should behave identically for all targets

Put code in a version module when:

- It touches Minecraft APIs that differ between versions
- It touches loader APIs
- It uses mixins targeting version-specific class or method signatures
- It is registration, entrypoints, or event wiring
- It is version-specific metadata or resources

Rule of thumb

- If a change would only apply to one Minecraft version, it probably belongs in that version module.
- If a change should apply everywhere, it belongs in `common/`.

## Step-by-step: add a new Minecraft version module

This assumes you are adding a new version under an existing loader, for example Forge.

### 1) Create the module folder

Copy the closest existing module.

Example target:
- From: `forge/mc_1_20_2`
- To: `forge/mc_1_21_1`

Keep the new module as similar as possible at first.

### 2) Wire the module into Gradle

- Add the new module to `settings.gradle` includes
- Ensure it depends on `common` (and any loader-common module if present)
- Ensure the module produces a jar with a unique classifier including loader + mc version

Expected outcome
- Gradle sees the module
- The module appears in `./gradlew projects`

### 3) Update the build configuration for the new target

In the new module’s `build.gradle` (or shared gradle conventions if used), update:

- Minecraft version
- Forge/NeoForge/Fabric version
- Mappings configuration
- Any dependency versions that are tied to the Minecraft version

Do not update old modules as part of this change unless required to keep the build working.

### 4) Fix compilation errors with a strict strategy

Your goal is to keep the new module thin. When you hit errors:

- If the broken code is version-specific glue, fix it in the new module only
- If the broken code is shared logic, move or refactor it into `common`

Avoid copying big chunks from old modules into the new one.

### 5) Update mixins cautiously

Mixins are a common source of version-specific breakage.

Guidelines
- Keep mixin configs inside the version module
- Prefer fewer mixins and smaller targets
- If a mixin target signature changes, fix it only in the affected module
- Document the change in the module’s notes or in the tracker issue

### 6) Confirm resources and metadata

For Forge/NeoForge, ensure `META-INF/mods.toml` exists and matches the target.

If a resource format changes between Minecraft versions:
- keep version-specific variants in the version module
- keep shared assets in `common` only when safe

### 7) Run the minimum smoke test

At minimum
- launch the dev client for the new module
- create/load a world
- place and configure both blocks

Capture evidence
- if it fails, attach logs to the issue
- if it works, update the version matrix

### 8) Update docs/versions.md

- Add or update the row for the new target
- Set status to Beta or Stable
- Add one sentence note if anything is missing
- Link the tracking issue

## Testing checklist (copy into the
