# Happy HTTP Mod Use Cases

## Overview
Happy HTTP Mod links Minecraft redstone with real-world HTTP traffic. It ships an embedded HTTP server and two configurable blocks:
- **HTTP Receiver Block** listens for webhook calls at custom URLs and powers redstone when incoming parameters match the block's configuration.
- **HTTP Sender Block** emits HTTP GET or POST requests with configured parameters whenever it receives a redstone signal.

Together, these pieces let in-game actions control external services and allow outside triggers—QR codes, smart-home sensors, web apps—to influence in-game builds.

## Core mechanics that enable integrations
- Incoming webhooks power receiver blocks via an endpoint-specific handler. Each block can demand exact key/value pairs and optionally use timers to keep power on for a defined duration, letting you gate redstone outputs behind HTTP requests that carry the right clues or tokens.
- The HTTP server registers receiver handlers at runtime, exposing both local and public addresses so players can aim external tools, scanners, or automation platforms at the correct endpoints.
- Sender blocks translate redstone events into HTTP calls, supporting GET with query parameters or POST with JSON-style payloads to drive downstream APIs, message brokers, or automation services.

## Example use cases
### 1) Smart home crossover
Use receiver blocks as in-game doorbells that light redstone lamps when a front-door sensor posts to the correct endpoint, or wire sender blocks to toggle real smart lights when a base defense lever is pulled.

### 2) Server-wide alerts and logging
Connect sender blocks to webhooks for Discord, Slack, or logging APIs so that raid alarms, boss kills, or base intrusions automatically push notifications with contextual data from the triggered block.

### 3) Redstone puzzle locks and ARG events
Require players to discover and submit specific parameter combinations to a receiver URL before a door powers on. Timed powering lets you force quick coordination or simultaneous switches during events or collaborative raids.

### 4) QR-powered treasure hunt (inside and outside Minecraft)
Run a multi-stage treasure hunt that mixes digital and physical clues:
- Place QR codes in the real world that post to receiver endpoints when scanned, opening hidden in-game passages powered by receiver blocks.
- Hide new clues behind those doors; when players find and activate them, sender blocks fire HTTP requests to external APIs that trigger the next automation step—opening a real-world smart lock, sending SMS/Discord hints, or updating a live scoreboard.
- Because receiver filters can check for specific parameters, each QR or clue can carry unique tokens, ensuring doors only open when the right physical clue is scanned, while sender payloads can pass player metadata to orchestrate bespoke downstream actions.

### 5) Education and workshops
Demonstrate web fundamentals by letting students see how HTTP parameters, redirects, and methods affect redstone outcomes. Assign tasks to build circuits that only activate when a lesson-specific payload is submitted, or to call classroom services via sender blocks.
