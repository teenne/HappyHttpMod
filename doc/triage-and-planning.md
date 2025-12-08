# docs/triage-and-planning.md

This document defines how we triage issues, how we plan work, and how we avoid duplicate “compatibility” issues for every Minecraft patch version.

It is written for a project with a product owner (priorities and roadmap) and technical contributors (implementation).

## What you already have

You already have:
- Issue templates (bug, compatibility, docs, feature, performance, support)
- A security reporting path
- Labels for loaders, client/server, and many Minecraft versions

This is good, but without rules it will produce noise and scattered progress.

## Source of truth

- Compatibility status lives in `docs/versions.md`
- Work items live in GitHub Issues and PRs
- Discord is for quick coordination, not the permanent record

If a decision affects code or compatibility claims, it must be recorded in GitHub.

## Triage roles

### Project owner

Responsible for:
- Setting priorities and roadmap
- Deciding what to work on next
- Closing or de-prioritising issues that do not fit current direction
- Keeping the documentation structure clean and consistent

### Technical contributors

Responsible for:
- Making issues actionable (asking for logs, reproductions)
- Picking issues and delivering PRs
- Keeping PR scope small and testable
- Updating `docs/versions.md` when support status changes

## Issue types and where they belong

Use the existing templates.

- Bug report: something is broken, with steps and logs
- Compatibility report: a specific loader + MC version fails to build, launch, or function
- Documentation request: missing or unclear docs
- Feature request: new behaviour, UX, or capabilities
- Performance issue: lag, CPU, spam, memory, etc.
- Support request: how-to questions (prefer Discord, unless it exposes a docs gap)

## Compatibility policy (important)

We do not want one open issue per patch version unless there is a real, evidenced break.

Rules:
- If you do not have a failure and logs, do not open a compatibility issue
- If the question is only “does it work on X”, update `docs/versions.md` to **Untested** and comment in the tracker
- Patch versions are assumed compatible within a supported minor line unless a patch introduces a known break

What we track as separate compatibility issues:
- Build fails
- Crash on launch or world load
- Functional regression in blocks, config, or networking

What we do not track as separate issues:
- “Compatibility with 1.21.9” with no failure details

## Labels (how we use them)

### Required labels for actionable issues

For bugs and compatibility:
- One loader label: `loader-forge` or `loader-neoforge` or `loader-fabric`
- One environment label: `Client` and/or `Server`
- One platform label: `mc-java` (and `mc-bedrock` only if that is ever relevant)
- One version label:
  - Prefer `version-1.21` plus a note for patch specifics, unless the patch is uniquely broken
  - If a patch is uniquely broken, use the patch label

For feature requests:
- `enhancement`
- Optional: loader labels if it is loader-specific

For docs:
- `documentation`

For support:
- `question` (and consider closing once answered)

### Priority labels

Priority is set by the project owner.
If you use priorities, keep it simple:
- `priority-high`
- `priority-medium`
- `priority-low`

## Tracker issues (how we plan bigger efforts)

We use tracker issues for broad efforts, with a checklist and links to child issues.

Examples:
- Forge 1.21.x compatibility tracker
- Multi-module Gradle refactor tracker
- Multi-loader architecture tracker

Tracker rules:
- The tracker is pinned while active
- All patch-version issues should link back to the tracker
- The tracker links to `docs/versions.md` and the porting guide
- The tracker defines what “done” means for the milestone

## Closing rules

Close issues when:
- It is a duplicate (link the canonical issue or tracker)
- It is a support question that belongs in Discord and is answered
- It lacks logs and the reporter does not respond after a reasonable time
- It is out of scope for current roadmap (use `wontfix` with a short reason)

## Minimal “done” definitions for planning

### Compatibility port done

- Target module builds
- Dev client launches
- World loads
- Receiver and sender pass the smoke test
- `docs/versions.md` updated to Beta or Stable
- Release notes updated when a release is cut

### Feature done

- Feature implemented
- Basic test notes included in PR
- Docs updated if user-facing behaviour changed

## Workflow from Discord to GitHub

Use Discord for speed, then capture the result:

- If a decision is made on Discord, post a short summary comment in the relevant issue
- If a bug report arrives on Discord, ask them to use the Bug Report template with logs
- If someone volunteers to help, ask them to comment on the tracker issue with what they will take

## What we will change in the issue list

If the repo currently has one compatibility issue per patch version:
- We will keep one tracker issue per minor line (for example Forge 1.21.x)
- We will close or consolidate patch issues that contain no failure details
- We will keep only issues that describe a specific break with logs
- The versions matrix becomes the visible “status page” for the community
