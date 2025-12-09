# HTTP Block Settings Reference

This document explains every configurable option for the **HTTP Receiver** and **HTTP Sender** blocks, including how parameter maps can be used to identify players, teams, and access levels. Example payloads are provided to illustrate typical setups.

## HTTP Receiver Block
The Receiver listens for POST webhooks at a configurable endpoint and powers redstone when incoming parameters match its rules.

### Endpoint
- **URL Path** (`url`): Relative path (e.g., `/secret/door`) the block registers on the built-in webhook server. Missing a leading `/` is automatically corrected.
- **Displayed Addresses**: The settings screen shows the internally bound address and the public address of the webhook server for quick copy/paste when sharing with external services.

### Powering Behavior
- **Power Mode** (`poweredType`):
  - **Switch** – toggles the block on/off each time a valid request is received.
  - **Timer** – powers the block for a fixed duration after a valid request.
- **Timer Duration** (`timer` + `timerUnit`):
  - Duration can be expressed in **ticks** or **seconds**.
  - When in Timer mode, powering resets on each valid request and automatically turns off after the configured duration.

### Redirects
- **Client Redirect** (`redirectClientUrl`): Optional URL sent back to the caller via HTTP 308 after processing. Useful for sending the requester to a confirmation page or error handler.

### Parameter Matching
- **Parameter Map** (`parameterMap`): Key/value pairs that must be present in the request. All entries must match exactly; if the map is empty, any request is accepted.
- Parameters can be supplied in the query string or request body; both are read by the handler.

#### Identity & Access Use Cases
- **Player targeting**: Require `player=Alex` to only react to Alex’s webhooks.
- **Team routing**: Use `team=red` so only Red Team requests power the block.
- **Access levels**: Combine parameters such as `role=admin` and `token=XYZ` to gate the signal to privileged callers.

#### Example Receiver Scenarios
- **Base entry scanner**
  - Endpoint: `/base/entry`
  - Power Mode: `Timer` for `10 seconds`
  - Parameters: `team=blue`, `access=level2`
  - Result: Redstone stays powered for 10 seconds when Blue Team sends a Level 2 access ping.
- **Secret door toggle**
  - Endpoint: `/secret/door`
  - Power Mode: `Switch`
  - Parameters: `player=ElytraMaster`
  - Result: Door toggles every time ElytraMaster’s QR code triggers the webhook.
- **Event badge check-in**
  - Endpoint: `/event/checkin`
  - Power Mode: `Timer` for `5 ticks`
  - Parameters: `badge_id=1234`, `role=staff`
  - Redirect: `https://intranet/event/ok`
  - Result: Short redstone pulse for staff check-ins and browser is redirected to the intranet confirmation page.

## HTTP Sender Block
The Sender issues HTTP requests when it receives a redstone signal.

### Target
- **URL** (`url`): Full destination URL that will receive the request.

### HTTP Method
- **Method** (`httpMethod`): Choose between **GET** or **POST**.

### Parameters
- **Parameter Map** (`parameterMap`): Key/value pairs appended as query parameters for GET or sent as a JSON body for POST.

#### Identity & Access Use Cases
- **Notify team services**: Send `team=alpha` and `status=alert` to a Discord webhook when an alarm triggers.
- **Player status reporting**: POST `{ "player": "Steve", "location": "spawn" }` to an external dashboard when a sensor fires.
- **Access logging**: Include `gateway=main_gate` and `level=3` to log which area triggered the signal.

#### Example Sender Scenarios
- **Trigger smart lights**
  - URL: `https://smarthome/api/lights`
  - Method: `POST`
  - Parameters: `room=lab`, `state=on`, `trigger=redstone_button`
  - Result: Pressing the button in-game turns on lab lights via the home automation API.
- **Scoreboard update**
  - URL: `https://game.example.com/score`
  - Method: `GET`
  - Parameters: `team=yellow`, `points=5`
  - Result: Redstone pulse updates the external scoreboard for the Yellow team.
- **Security alert webhook**
  - URL: `https://hooks.example.com/security`
  - Method: `POST`
  - Parameters: `player=Intruder42`, `zone=treasury`, `severity=high`
  - Result: Tripwire triggers a POST that pages admins with precise context.

## Tips
- Keep Receiver endpoints unique per behavior to avoid accidental overlap.
- When stacking multiple Receivers on the same endpoint, the last configured redirect URL is used for responses.
- Use small Timer durations (ticks) for short pulses or longer seconds-based timers for sustained power.
- Validate Sender URLs and parameters in a browser or API client before wiring them to redstone to simplify debugging.
