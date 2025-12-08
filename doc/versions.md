# Supported versions

This document is the source of truth for which combinations of loader and Minecraft version are supported.

If you are about to open an issue titled “Compatibility with Minecraft X”, read this first. In most cases, the right next step is to update this matrix and link to a single compatibility tracking issue.

## How to read this

Status meanings

- **Stable**: works for normal play, basic testing completed
- **Beta**: builds and launches, but needs more testing
- **Broken**: known to fail (build error, crash, or major feature broken)
- **Untested**: no confirmed status yet

Notes

- “Supported” means the mod builds and runs, and both blocks work in a basic smoke test.
- We do not treat every patch version as a separate target unless it introduces a real break. If there is no known breakage, a patch version is assumed to work when the minor line is supported.

## Compatibility matrix

| Loader | Minecraft version | Status | Notes | Tracking issue |
| --- | --- | --- | --- | --- |
| Forge | 1.20.2 | Stable | Current working baseline | (link) |
| Forge | 1.20.3–1.20.6 | Planned | Not started | (link) |
| Forge | 1.21.0–1.21.10 | Planned | Not started | (link) |
| NeoForge | 1.21.x | Planned | Not started | (link) |
| Fabric | TBD | Planned | Not started | (link) |

Replace “(link)” with the GitHub issue link(s) once the tracker issue exists.

## Current focus

- Primary target: **Forge 1.21.10** (port from Forge 1.20.2)
- Secondary: stabilise the multi-module build so multiple versions can be built from one repo

## What counts as “works”

Minimum smoke test checklist

- Game launches to main menu
- World loads (single-player)
- HTTP Receiver block can be placed and configured
- HTTP Sender block can be placed and configured
- Receiver triggers redstone as expected on request
- Sender sends a request on redstone input
- Config file generates correctly and settings take effect

If any of the above fails, the status should be **Broken** with a short note and a link to logs.

## How to update this file

When you verify a version, update the matrix row with:

- Status (Stable, Beta, Broken, Untested)
- Notes (one sentence, include the breakage area if Broken)
- Tracking issue link (compatibility tracker or specific bug)

Keep notes short and factual.

## When to create a new compatibility issue

Create a new issue only when you have one of these:

- A specific build error for a specific version
- A runtime crash with a log/crash report
- A functional regression (block behaviour, config, networking)

If you are only asking “does it work on version X”, update the matrix status to **Untested** and add yourself as the tester in the tracking issue instead of opening a new standalone issue.

## Java requirements

Java requirements vary by Minecraft version and loader.

Until we pin the exact requirements here, follow the loader documentation and the module’s Gradle configuration. This section should be updated as we add more version modules.
