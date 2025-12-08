# docs/porting-guide.md

This guide explains how to port Happy HTTP to a new Minecraft version for a given loader.

It is written for a repository that builds one jar per loader + Minecraft version module, with shared logic in `common/`.

## Porting strategy

We use a two-layer approach:

- `common/`: shared logic, no Minecraft or loader API imports
- `<loader>/mc_<version>/`: version-specific glue, registrations, and resources

Porting should mostly be mechanical changes in the glue layer.

If you find yourself rewriting large parts of shared logic, stop and document why in the issue or PR.

## Before you start

1) Check the support matrix
- `docs/versions.md`

2) Check the active tracker issue
- For example the Forge 1.21 tracker

3) Pick one target
- One loader
- One Minecraft version line
- One module at a time

4) Keep PR scope small
- One version module per PR is the default
- Avoid mixing ports with new features

## Step-by-step port workflow

### Step 1: create the target module

Create a new module under the loader folder.

Example
- `forge/mc_1_21_1/`

Copy only what is needed from the baseline module, then adjust:
- Minecraft version
- Forge version (and mappings)
- mod metadata as required by the loader

Ensure jar naming includes loader and Minecraft version.

### Step 2: make it compile

Run:

```sh
./gradlew :forge:mc_1_21_1:build
```

Fix compilation errors in the module.

Rules
- Do not “fix” compile errors by duplicating shared logic
- If a change belongs in `common/`, move it there only if it is truly shared and does not require Minecraft APIs

### Step 3: get to main menu

Run the dev client for the module (task names vary by setup). Your goal is:
- game reaches main menu without crashing

If it crashes:
- capture `latest.log` and crash report
- link them in the tracking issue
- commit minimal fixes that make progress

### Step 4: world load and basic placement

- Create a new world
- Ensure the mod loads and blocks appear in creative inventory
- Place each block without crashing

### Step 5: functional smoke test

Minimum smoke test:

HTTP Receiver
- Place block
- Configure endpoint
- Send an HTTP request
- Verify it triggers redstone output

HTTP Sender
- Place block
- Configure target URL and method
- Trigger with redstone
- Verify request is sent

Config
- Confirm `happyhttpmod-config.toml` is created
- Confirm binding behaviour and port settings apply

### Step 6: update docs and tracking

- Update `docs/versions.md` status (Untested → Beta when smoke test passes)
- Link the PR in the tracker issue
- If you found a repeatable breakpoint, open or update a compatibility issue with logs

## How to handle common breakpoints

### Renamed or moved Minecraft classes

Fix in the version module first.

Only touch `common/` if the shared interface needs to change and you can keep it free of Minecraft imports.

### Registry and initialisation changes

Loader APIs and lifecycle ordering change between versions.

Prefer to:
- keep registration and init in the version module
- expose only stable “hooks” into common logic

### Networking and screens

UI and networking are often version-sensitive.
Keep those components version-local.

### HTTP server lifecycle

Ensure the HTTP server:
- starts at a safe time
- does not block the game thread
- stops cleanly on shutdown
- binds predictably to configured IP/port

If lifecycle differs across versions, implement the lifecycle in the version module and keep request handling shared.

## What to include in your PR

- Which target module you ported
- Build command that succeeds
- Smoke test results (or which part fails)
- Logs/crash reports if you fixed a crash
- Any doc updates (especially `docs/versions.md`)

## When to stop and ask for alignment

Stop and write into the issue if:
- you need a cross-cutting architectural change
- you want to introduce a new dependency used across modules
- you think the module structure plan needs to change

Keep changes intentional and recorded.

## Linked docs

- Repo layout rules: `docs/repo-layout.md`
- Testing: `docs/testing.md`
- Build and release: `docs/build-and-release.md`
- Versions matrix: `docs/versions.md`
