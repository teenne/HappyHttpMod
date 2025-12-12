# Happy HTTP Mod – Architectural and Security Review

## Overview
The mod exposes in-game triggers and HTTP interactions to external systems. This review summarizes documentation gaps, architectural risks, and security concerns, and provides concrete improvements to align the implementation with the stated roadmap and user expectations.

## Documentation Gaps & Mismatches
- **Config format mismatch.** README describes `happyhttpmod-config.json` with `webhookServerIP`/`webhookServerPort`, but Forge implementation reads a Forge config plus `happyhttp-global-vars.toml` written as Java properties using keys like `port`, `local_adress`, and `global_param*`.
  - **Improve:** Unify on a single config schema (suggest TOML or JSON), document the actual keys, and add validation/auto-migration when formats change.

## Architectural / Reliability Concerns
- **HTTP server startup robustness.** Server starts synchronously during `onServerStarting`; failures only log stack traces, potentially leaving handlers unavailable without clear signals.
  - **Improve:** Start the HTTP server asynchronously with bounded timeouts, surface status in logs and a game chat toast, and fail-fast with clear error messages when binding fails.
- **External IP discovery blocking.** Uses `http://checkip.amazonaws.com` on every start, blocking main thread without timeout and leaking failures to logs.
  - **Improve:** Move IP discovery to a background task with a short timeout and cache, and provide a config flag to disable the call.
- **Handler collision and URL hygiene.** Handler registration overwrites contexts on URL collisions; URLs are minimally validated (leading slash only), risking silent conflicts.
  - **Improve:** Normalize paths, reject duplicates with explicit errors, and add unit tests for handler registration.
- **Fragile request parsing.** Parameter parsing assumes valid JSON bodies and processes GETs even though `ALLOWED_METHOD` is POST; malformed payloads can throw unchecked exceptions.
  - **Improve:** Enforce HTTP method checks, return 405 on mismatch, add input size limits, and respond with structured error JSON for parse failures.
- **Outbound request feedback.** Client POST/GET wrappers swallow responses or exceptions, always returning empty strings.
  - **Improve:** Return status codes/body or throw checked exceptions; add retry/backoff hooks so calling code can react to failures.

## Security Issues
- **Unauthenticated, plaintext HTTP.** Uses `com.sun.net.httpserver.HttpServer` without TLS or authentication, exposing redstone triggers to any reachable host. Default bind address `192.168.0.1` increases unintended LAN exposure.
  - **Improve:** Default-bind to localhost, add optional TLS (self-signed or provided certs), and support token-based auth or IP allowlists. Document the risks prominently.
- **Open redirect and abuse potential.** Redirect URLs are stored as plain text and returned with HTTP 308 without rate limiting or validation.
  - **Improve:** Validate redirect targets against an allowlist/regex, cap redirect frequency per client/IP, and add logging/metrics for abuse detection.
- **External IP leak without consent.** Startup IP lookup discloses server presence to third-party service by default.
  - **Improve:** Make external calls opt-in, document the endpoint, and log when/if the request is made.

## Roadmap Blockers & Recommendations
- **Multi-loader support.** With Fabric/NeoForge modules disabled, promised multi-loader support is undeliverable.
  - **Improve:** Re-enable modules in `settings.gradle`, fix loader-specific code paths, and add CI matrices for Forge/Fabric/NeoForge. If not feasible, adjust roadmap to Forge-only.
- **Configuration governance.** Without schema validation and migration, deployments will be brittle.
  - **Improve:** Introduce a versioned config schema, provide defaults, and add a `--validate-config` server command for administrators.
- **Testing & observability.** Lack of automated tests and observability makes regressions likely.
  - **Improve:** Add unit tests for handler registration and request parsing, integration tests for HTTP lifecycle, and structured logging with per-request correlation IDs.

## Quick Wins
- Default the bind address to `127.0.0.1` and add a config flag for LAN exposure.
- Add method enforcement and JSON parse guards to receiver endpoints.
- Document the actual configuration files and keys currently used by the Forge build.
- Surface HTTP server status in logs and to in-game administrators during startup.
