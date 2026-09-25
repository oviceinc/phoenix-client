# phoenix-client

Maintained copy of the [Phoenix](https://github.com/phoenixframework/phoenix) JavaScript client, based on **1.7.14**.

The official npm package is [`phoenix`](https://www.npmjs.com/package/phoenix). This repository is **not** a drop-in replacement for current 1.8.x; it keeps oVice-specific behavior that upstream still does not provide.

## Differences from upstream

Compared with `phoenixframework/phoenix` (JS client / npm `phoenix` 1.8.x):

- **Reconnect on WebSocket close code 1000.** Upstream skips reconnect when the close code is 1000 (`closeCode !== 1000`). This fork reconnects on 1000 and uses close code `3000` (`WS_CLOSE_HEARTBEAT_TIMEOUT`) for heartbeat timeouts, because browsers and networks sometimes close with 1000 even when a reconnect is required.
- **Separate heartbeat interval and timeout.** Adds `heartbeatTimeoutMs`. Upstream uses `heartbeatIntervalMs` for both the send interval and the timeout.
- **Ghost reconnect timer during teardown.** The reconnect timer sets `closeWasClean = true` before `teardown()`, so `onConnClose` does not schedule another reconnect. Under load this otherwise cycles connections before they open.
- **WebSocket leak when multiple teardowns run.** Teardown captures the connection being closed and ignores later close/cleanup on a replaced socket. Upstream 1.8.x has a similar `connToClose` fix.
- **Bundled TypeScript types.** Ships `js/phoenix/index.d.ts`. Official npm `phoenix` does not include types (`@types/phoenix` is a separate package).
- **Package exports and `global` initialization.** ESM `exports` point at `js/phoenix`, and `global` is accessed in a way that avoids `ReferenceError: Cannot access 'global' before initialization`.

An upstream sync was attempted and reverted (PR #4 / #6) because newer upstream behavior (`pageHidden` reconnect pause, heartbeat close code 1000, `closeWasClean` changes) conflicts with the items above.
