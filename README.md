# OpenClaw Chrome Extension (Browser Relay)

Purpose: attach OpenClaw to an existing Chrome tab so the Gateway can automate it (via the local CDP relay server).

## Dev / load unpacked

1. Build/run OpenClaw Gateway with browser control enabled.
2. Ensure the relay server is reachable at `http://127.0.0.1:18792/` (default).
3. Install the extension to a stable path:

   ```bash
   openclaw browser extension install
   openclaw browser extension path
   ```

4. Chrome → `chrome://extensions` → enable “Developer mode”.
5. “Load unpacked” → select the path printed above.
6. Pin the extension. Click the icon on a tab to attach/detach.

## Options

- `Relay port`: defaults to `18792`.
- `Gateway token`: required. Set this to `gateway.auth.token` (or `OPENCLAW_GATEWAY_TOKEN`).

```mermaid
sequenceDiagram
    participant User
    participant OptionsPage
    participant Background
    participant ChromeAPI
    participant OpenClawApp

    Note over User, OpenClawApp: PHASE 1: Setup and Authentication
    User->>OptionsPage: Types in Port and Secret Token
    OptionsPage->>Background: Test this connection
    Background->>OpenClawApp: HTTP GET /json/version
    OpenClawApp-->>Background: 200 OK (Valid Server)
    Background-->>OptionsPage: Success
    OptionsPage->>ChromeAPI: Save Port and Token
    
    Note over User, OpenClawApp: PHASE 2: Activating the Bridge
    User->>Background: Clicks Extension Icon
    Background->>ChromeAPI: Load Port and Token
    Background->>Background: Do Crypto Math (HMAC)
    Background->>OpenClawApp: Connect via WebSocket
    OpenClawApp-->>Background: Connection Established
    Background->>ChromeAPI: chrome.debugger.attach(Tab)
    ChromeAPI-->>Background: Debugger Hooked
    Background->>OpenClawApp: Tab is ready! (Session ID)
    Background->>User: Changes Badge to ON
    
    Note over User, OpenClawApp: PHASE 3: Remote Control
    OpenClawApp->>Background: Send CDP Command
    Background->>ChromeAPI: chrome.debugger.sendCommand
    ChromeAPI-->>Background: Result of command
    Background->>OpenClawApp: Send Result back
    
    Note over User, OpenClawApp: PHASE 4: Surviving a Page Load
    User->>ChromeAPI: Clicks a link to a new website
    ChromeAPI-->>Background: SECURITY Detached!
    Background->>OpenClawApp: Hold on we lost the tab
    Background->>User: Changes Badge to Connecting
    
    loop Retry Loop
        Background->>ChromeAPI: Try to re-attach debugger
    end
    
    ChromeAPI-->>Background: Debugger Hooked (New page)
    Background->>OpenClawApp: We are back!
    Background->>User: Changes Badge back to ON
```