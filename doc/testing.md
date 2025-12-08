# docs/testing.md

This document describes how we test Happy HTTP across multiple loaders and Minecraft versions, and what evidence is required before we mark a target as supported.

It is designed for a repository where each jar is built by one module (loader + Minecraft version).

## Principles

- Keep tests lightweight and repeatable
- Prefer a clear smoke test that catches the common failures
- Record results in `docs/versions.md`
- When something fails, capture logs and link the issue

## Test levels

### Level 1: build verification

Goal
- The target module compiles and produces a jar

Commands (examples)

```sh
./gradlew :forge:mc_1_21_1:build
```

Expected result
- Build succeeds
- Jar exists in `<module>/build/libs/`
- Jar name includes loader and Minecraft version

### Level 2: launch verification

Goal
- The dev client launches for the target module

Expected result
- Game reaches main menu without crashing
- No critical errors during initialisation

Evidence to capture if it fails
- `latest.log`
- crash report (if present)
- short description of when it crashes (during init, after world load, on placing block)

### Level 3: functional smoke test (minimum for Beta)

Goal
- Both blocks can be used and perform their core function

Checklist
- [ ] Dev client launches
- [ ] Create or load a world
- [ ] Place HTTP Receiver block
- [ ] Open its UI and set endpoint
- [ ] Send a request to the configured endpoint
- [ ] Verify receiver triggers redstone
- [ ] Place HTTP Sender block
- [ ] Configure target URL and method
- [ ] Trigger it with redstone
- [ ] Verify request was sent (confirm via your local server logs or a request inspector)
- [ ] Quit and relaunch
- [ ] Verify config persists and still works

Minimum status rule
- If Level 1 and Level 2 pass, status can be **Beta** once Level 3 passes
- **Stable** requires repeatability and at least one additional sanity pass (see below)

## Recommended test tools

Any of these are acceptable. Use what is easiest.

- Local HTTP receiver
  - a simple local server that logs requests
- Request inspector services
  - use only if you are comfortable sending test requests to a third party

For Receiver testing
- Use `curl` to hit the endpoint and include parameters

Example

```sh
curl "http://<server-ip>:<port>/<endpoint>?a=1&b=2"
```

For Sender testing
- Point the Sender to a local endpoint you control and verify it receives requests

## Server testing (optional but recommended)

If the module is intended for servers, confirm basic behaviour on a dedicated server.

Checklist
- [ ] Server starts with the mod installed
- [ ] Client can connect
- [ ] Blocks place and function
- [ ] Webhook server binds correctly (as configured)
- [ ] No repeated errors or spam in logs

## Regression testing when changing common/

If you change `common/`, you can break multiple targets without noticing.

Minimum expectation
- `./gradlew buildAll` succeeds

Recommended if possible
- run the Level 3 smoke test on at least:
  - the baseline target (currently Forge 1.20.2)
  - the newest supported target

## Recording results

When you finish testing, update:

- `docs/versions.md` status and notes

If the target passes
- set to **Beta** or **Stable**
- add one sentence note if needed
- link the relevant issue or PR

If the target fails
- set to **Broken**
- add one sentence describing the failure area
- link the issue with logs

## What to include in bug reports from testing

- Loader and Minecraft version
- Mod version or commit hash
- Whether it is client, server, or both
- Exact steps to reproduce
- `latest.log` and crash report
- Any relevant configuration used (`happyhttpmod-config.toml`)

## What not to do

- Do not mark a version as supported just because it compiles
- Do not open one issue per patch version unless you can show a real break
- Do not rely on Discord messages as the only place test outcomes are recorded

Keep the matrix updated. That is how contributors and users will know what works.
