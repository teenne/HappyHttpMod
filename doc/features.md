# docs/features.md

This document turns feature requests into implementable specs and clarifies dependencies and release order.

It is owned by the project owner. Technical contributors should use it to plan PR scope and avoid building overlapping systems twice.

## Prioritisation model

We group features by value and dependency.

- Tier 0: compatibility and structure (must be stable first)
- Tier 1: reduce setup friction and support load
- Tier 2: improve builder experience and debugging
- Tier 3: advanced gameplay mechanics

This is not a promise. It is a planning tool.

## Feature specs

### #37 Port binding helper (server diagnostics)

Status
- Planned

Tier
- Tier 1 (setup friction reduction)

Problem
- Users struggle with local IP selection, port conflicts, NAT and port forwarding, and basic reachability testing. This causes support load and drop-off.

Proposed UI entry points
- Button inside HTTP Receiver UI: “Server diagnostics”
- Optional: command for headless servers

Core capabilities
- Interface/IP selection UI with “Auto” and explanation
- Port bind test and clear error messages
- Local reachability summary (exact local URL)
- External reachability guidance checklist (docs link)
- Optional UPnP mapping (default off, with security warning)

Data and safety constraints
- Never print secrets in logs
- Do not attempt invasive network scans
- Clear separation between “what we can detect” and “what we cannot”

Dependencies
- None, but benefits from the debugging UI improvements in #26 and #27

Definition of done
- A new diagnostics UI exists and helps users select IP and port correctly
- It reports port binding conflicts clearly
- It provides a test URL and a short, explicit checklist for LAN and WAN reachability

Notes
- Keep the first version minimal. Avoid building a full network tool inside Minecraft.

---

### #25 Reusable configuration presets (“blueprints”)

Status
- Planned

Tier
- Tier 2 (builder speed and consistency)

Problem
- Builders repeat URLs and parameters across blocks, causing copy/paste errors and inconsistent adventure map setups.

Core UX
- Preset dropdown in Receiver and Sender UIs
- Actions
  - Save current as preset
  - Apply preset
  - Delete preset
  - Optional: import/export

Preset contents

Receiver should capture
- Endpoint path
- Expected query parameters key/value
- Redirect URL
- Power type and pulse duration
- Local vs external endpoint selection
- Any auth fields when #31 exists

Sender should capture
- Target URL
- Method (GET/POST)
- Parameters/body
- Headers/auth when #31 exists

Storage strategy (phased)
- Phase 1: per-world presets (stored in world save)
- Phase 2: per-player presets (optional)
- Phase 3: datapack-shippable presets for adventure maps (optional)

Dependencies
- Depends on stable config/model layer (common module cleanliness)
- Strongly benefits from #26 resolved endpoint preview
- Auth integration depends on #31

Definition of done
- Presets can be saved and applied reliably
- Presets are size-limited and do not bloat saves
- A map creator can ship a world with presets pre-configured

---

### #36 Adventure map toolkit (examples and templates)

Status
- Planned

Tier
- Tier 2 (onboarding and adoption)

Problem
- Users understand the idea but struggle to design robust builds. Working examples teach faster than text.

Deliverables (phased)
- Phase 1: companion download (world or datapack) with examples
- Phase 2: structure files usable with structure blocks
- Phase 3: optional in-game “load examples” helper

Example set (initial)
- Receiver-triggered hidden door
- Receiver-triggered alarm that calls Sender to external webhook
- Escape room lock that combines real-world trigger plus in-game action

Documentation
- Add “Examples” section to README
- Provide step-by-step import and test flow

Dependencies
- Works best after #25 presets exist, but can start earlier as a download
- Benefits from #37 diagnostics for easy setup

Definition of done
- A user can import examples and see them working with clear instructions
- The examples are maintained across supported versions

---

### #27 Inline testing and per-block logs

Status
- Planned

Tier
- Tier 1 or Tier 2 (debugging, support reduction)

Problem
- Builders cannot validate config without reading server logs.

Sender requirements
- Test/Ping button
- Result summary: timestamp, status, HTTP code, short error reason

Receiver requirements
- Last received request summary
- Whether it matched expected parameters

Shared
- Per-block log in UI (last N entries, bounded)
- Clear log
- Save-safe storage limits

Dependencies
- None, but aligns well with #26 and #33

Definition of done
- Users can validate sender/receiver behaviour inside Minecraft
- Logs are bounded and do not bloat world saves

---

### #26 Visual connection cues and clearer debugging information

Status
- Planned

Tier
- Tier 1 or Tier 2

Problem
- In complex builds, it is unclear what endpoint is configured and whether anything is happening.

Requirements
- Resolved endpoint preview in UI
- Parameter preview
- Optional activity cues (config toggle)

Dependencies
- Can be implemented independently
- Complements #27 and #33

Definition of done
- UI clearly shows the final resolved URL/endpoint
- Users can see whether last request matched or failed

---

### #33 Compact status indicators on block model

Status
- Planned

Tier
- Tier 2 (in-world debugging)

Problem
- In-world feedback is missing, builders cannot quickly identify failing blocks.

Requirements
- Subtle in-world state indicators for sender and receiver
- Defined states
  - Receiver bound OK
  - Activity pulse on receive/send
  - Last request failed
  - Optional auth error
- Config toggle to disable visuals

Dependencies
- Depends on internal state tracking (also used by #27)
- Auth error state depends on #31

Definition of done
- Builders can locate failing blocks without opening logs
- Indicators remain subtle and optional

---

### #32 Per-block rate limiting and cooldown

Status
- Planned

Tier
- Tier 2 (reliability and safety)

Problem
- Blocks can spam external services or jitter redstone builds.

Requirements
- Cooldown and rate limit controls for sender and receiver
- Behaviour on limit hit
  - ignore
  - optional single-slot queue
- Feedback in UI, and log entries when blocked

Dependencies
- Works best alongside #27 (so users can see blocked triggers)
- No hard dependency on other features

Definition of done
- A builder can protect against spam without complex redstone
- The behaviour is predictable and documented

---

### #31 Built-in authentication helpers

Status
- Planned

Tier
- Tier 1 or Tier 2 (security and safe defaults)

Problem
- WAN-exposed receivers need simple protection. Senders need auth helpers for common APIs.

Sender auth modes
- Bearer token
- Basic auth
- API key helper (header or query param)

Receiver auth modes
- Shared secret validation via query param and/or header
- Token generation helper
- Reject unauthorised requests (401/403) with no redstone output

Safety requirements
- Mask secrets in UI after saving
- Redact secrets in logs
- Keep data size bounded

Dependencies
- Should be aligned with presets (#25) so presets can carry auth configuration safely

Definition of done
- Common auth patterns work without external middleware
- Receiver can be exposed more safely for real-world triggers

---

### #28 Expand Receiver output modes

Status
- Planned

Tier
- Tier 2 (builder friendliness)

Problem
- Builders must build extra redstone for common timing patterns.

Output modes
- Toggle
- Pulse (duration)
- Timer (extendable or restartable)
- Repeating clock while active

Dependencies
- None, but interacts with rate limiting (#32) and receiver mapping (#34)

Definition of done
- Common patterns can be built without additional redstone components
- Behaviour is documented and testable

---

### #34 Receiver outputs based on content (payload mapping)

Status
- Planned

Tier
- Tier 3 (advanced logic)

Problem
- Receiver only gates on matching, not on producing multiple states.

Requirements
- Payload mapping section in Receiver UI
- Parameter key to redstone strength mapping
  - direct 0–15 clamp or scaled range
- Optional mapping table for named states
- UI shows last received value and last output

Dependencies
- Benefits from #27 and #26
- May overlap with filter block concept (#29), decide scope carefully

Definition of done
- One endpoint can represent multiple states reliably without requiring multiple blocks

---

### #35 Sender outcomes (response-driven gameplay)

Status
- Planned

Tier
- Tier 3 (advanced logic)

Problem
- Builders cannot branch based on external response.

Minimum viable scope
- Capture last status code and error type
- Output mode based on result (success/fail or mapped strengths)

Optional scope
- Advancement trigger
- Reward trigger
- Strictly bounded JSON key extraction, size-limited

Dependencies
- Strongly benefits from #27 logs
- JSON response parsing must be carefully bounded to avoid performance/save issues

Definition of done
- Sender can drive in-game logic based on response outcome without external middleware

---

### #30 Scene Sequencer block

Status
- Planned

Tier
- Tier 3 (complex adventures)

Problem
- Multi-step sequences require many blocks or complex redstone.

Requirements
- New block that executes a list of sender steps with delay and basic conditions
- Step editor UI: add/remove/reorder
- Per-step status reporting and minimal logs

Dependencies
- Depends on having a stable Sender implementation
- Strongly benefits from #27 logging and #25 presets

Definition of done
- A builder can run a multi-step sequence from one redstone input with predictable behaviour

---

### #29 HTTP Filter block

Status
- Planned

Tier
- Tier 3 (advanced validation and composition)

Problem
- Builders want whitelist/validate/transform without external middleware.

Key design decision
- Either extend Receiver (#34) or introduce a new block (#29), but avoid duplicating overlapping features.

Initial scope suggestion
- Start with Receiver mapping (#34) first
- Re-evaluate whether a separate Filter block is still needed afterwards

Dependencies
- Depends on clear data model and bounded storage
- Benefits from #27 and #33

Definition of done
- Advanced gating and mapping is possible without turning Receiver UI into a mess

## Cross-feature dependency map

- #37 Port helper improves onboarding for everyone, and reduces support load
- #26 + #27 are foundational debugging improvements
- #31 (auth) should be designed to integrate with #25 (presets)
- #32 (rate limiting) protects external services and stabilises builds
- #34 and #35 add richer gameplay mechanics after stability
- #30 and #29 are advanced and should not be started until core usability is good

