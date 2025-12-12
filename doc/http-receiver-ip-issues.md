# HTTP Receiver address display issues

This note summarizes why the HTTP Receiver GUI sometimes shows blank addresses on a local machine and why, on a multi-tenant server, it can display an internal IP like `192.168.0.1` that is not actually bound. It also outlines investigation steps and improvements to make the endpoint information reliable.

## How addresses are currently determined

* The HTTP server is created with `new InetSocketAddress(Services.HTTP_CONFIG.getPort())`, which binds to the wildcard address (all interfaces) but does **not** record which interface is preferred. After the bind, the code separately calls `getInternalIPv4Address()` and `getExternalIPAddress()` to populate the strings shown in the GUI.【F:common/src/main/java/com/clapter/httpautomator/http/HttpServerImpl.java†L33-L137】
* The internal address helper returns the **first** non-loopback IPv4 address it encounters, unless the `local_adress` config is set. If the config is populated, that value is returned verbatim without checking whether the HTTP server is actually bound to it.【F:common/src/main/java/com/clapter/httpautomator/http/HttpServerImpl.java†L49-L67】【F:forge/src/main/java/com/clapter/httpautomator/platform/config/HttpServerConfig.java†L28-L147】
* The external address helper makes an HTTP request to `http://checkip.amazonaws.com`; if that call fails (e.g., no Internet, blocked egress), the string stays empty.【F:common/src/main/java/com/clapter/httpautomator/http/HttpServerImpl.java†L69-L86】
* The GUI uses the addresses stored in the block entity values when the screen opens. Those values are filled by reading `CommonClass.HTTP_SERVER.getServerAddress()` and `getServerPublicAdress()` during packet encoding. If the HTTP server is not running, both helpers return an empty string, so the GUI shows blanks.【F:common/src/main/java/com/clapter/httpautomator/blockentity/HttpReceiverBlockEntity.java†L148-L193】【F:common/src/main/java/com/clapter/httpautomator/http/HttpServerImpl.java†L127-L137】【F:common/src/main/java/com/clapter/httpautomator/client/gui/HttpReceiverSettingsScreen.java†L93-L128】

## Why addresses can be blank on a local machine

* The display depends on the HTTP server being started before the GUI packet is encoded. If the server failed to start (port conflict, missing callback, or exception), `getServerAddress()` returns `""`, leaving the labels empty.【F:common/src/main/java/com/clapter/httpautomator/http/HttpServerImpl.java†L33-L137】
* External IP detection relies on reaching `checkip.amazonaws.com`; offline environments or blocked outbound traffic result in an empty string. There is no fallback or error message, so the GUI silently shows nothing.【F:common/src/main/java/com/clapter/httpautomator/http/HttpServerImpl.java†L69-L86】【F:common/src/main/java/com/clapter/httpautomator/client/gui/HttpReceiverSettingsScreen.java†L93-L128】
* If `local_adress` in the config is left empty, the helper picks the first interface. On machines where the first interface is a VPN adapter or an un-routed interface, this can resolve to nothing useful, and there is no validation that the address matches the bound socket.【F:common/src/main/java/com/clapter/httpautomator/http/HttpServerImpl.java†L49-L67】

## Why a multi-tenant server can show `192.168.0.1` that is not bound

* The Forge config defaults `local_adress` to `192.168.0.1`. If the operator does not change this, `getInternalIPv4Address()` returns that literal value even though the server bind uses the wildcard address and may not own `192.168.0.1` at all.【F:forge/src/main/java/com/clapter/httpautomator/platform/config/HttpServerConfig.java†L28-L147】【F:common/src/main/java/com/clapter/httpautomator/http/HttpServerImpl.java†L33-L67】
* Binding with only a port means the actual socket listens on all interfaces; the code never checks whether the configured `local_adress` is routable or part of the machine. The displayed IP can therefore disagree with what is reachable from the network, especially on hosts with many interfaces or containerized deployments.【F:common/src/main/java/com/clapter/httpautomator/http/HttpServerImpl.java†L33-L137】
* The GUI has no concept of multiple IPs. Only a single string is shown, so in multi-homed environments there is no way to choose the correct address for an external integration.【F:common/src/main/java/com/clapter/httpautomator/client/gui/HttpReceiverSettingsScreen.java†L93-L128】

## Suggestions for investigation and improvement

1. **Verify server startup flow**
   * Confirm the server start callbacks fire in both singleplayer and dedicated server modes. Log and surface errors when the HTTP server cannot bind to the configured port so the GUI is not populated with empty strings.
2. **Align displayed address with the actual bind**
   * Inspect `server.getAddress()` after binding; use the resolved host (`getHostString()` or the bound interface when not wildcard) and port for the internal address. If the socket is bound to wildcard, prefer enumerating interfaces and picking the address that matches the configured or preferred network.
3. **Improve internal address detection**
   * Change the default `local_adress` to blank, and only use a configured value after validating it exists on one of the machine interfaces.
   * When `local_adress` is empty, enumerate all non-loopback IPv4 addresses and either pick the primary one deterministically (e.g., highest priority, lowest metric) or present them to the user.
4. **Handle multiple interfaces in the UI**
   * Replace the static text with a dropdown populated from the server-reported interface list, highlighting the one the server is actually bound to. Provide both the reachable external URL and the chosen internal bind IP for clarity.
5. **Make external IP resolution resilient**
   * Cache the last successful external IP lookup and show a warning when it cannot be refreshed. Consider allowing a manual override in the config or UI for environments without outbound Internet access.
6. **Surface diagnostics to the player/operator**
   * Display status text in the receiver GUI when the HTTP server is offline, when address detection fails, or when the configured IP does not match any local interface. This reduces confusion when addresses otherwise appear blank.
