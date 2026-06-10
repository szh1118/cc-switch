<!-- claude-session-id: 66113990-09fb-40a7-91dc-7098effdb8b3; updated: 2026-06-10-15-10-00 -->

# Handoff

## Goal
Add complete LAN WebUI support to cc-switch with browser-based remote control.

## Current State — ALL COMPLETE ✅

### WebUI Feature Fully Implemented and Fixed
- **Backend HTTP API** (`src-tauri/src/webui.rs`, ~900 lines)
  - 50+ management endpoints (providers/proxy/settings/usage/models)
  - RFC 1918 private network auth: localhost + LAN no token, public IP requires token
  - Static file serving from `dist/` with SPA fallback
  - Auto-starts by default (opt-out via `CC_SWITCH_WEBUI=0`)
  - Dynamic start/stop/restart via Tauri commands

- **Frontend Browser Compatibility** (`src/lib/commandClient.ts`)
  - Runtime detection (Tauri vs browser)
  - Unified API abstraction (invoke → fetch)
  - Token handling for WebUI mode

- **Settings UI** (`src/components/settings/WebUiSettings.tsx`)
  - Enable/disable toggle
  - Port/host configuration
  - Token input (for public IP only)
  - Start/Stop/Restart controls
  - Access URL display with copy/open buttons
  - Updated warning: "允许同一网络访问（RFC 1918，无需令牌）"

### Recent Fixes (2026-06-10)
1. **Tauri devUrl removal** - Desktop UI now loads embedded frontend correctly
2. **RFC 1918 private network auth** - Localhost + LAN免token，only public IP needs token
3. **UI warning text** - Changed from amber "强制要求令牌" to blue "允许同一网络访问，无需令牌"

## Requirements Checklist
- [x] Backend HTTP API with smart authentication
- [x] Frontend browser compatibility layer
- [x] Auto-start WebUI by default
- [x] Serve static files from backend
- [x] Settings UI for WebUI configuration
- [x] Dynamic WebUI start/stop
- [x] Fix desktop UI loading (remove devUrl)
- [x] Fix WebUI 401 errors (RFC 1918 auth)
- [x] Update UI warning text
- [x] README documentation
- [x] PR submitted to upstream

## Key Files

### Backend
- `src-tauri/src/webui.rs` - WebUI server with RFC 1918 auth logic
  - `is_private_ip()` - Checks loopback + RFC 1918 + link-local
  - `start_from_settings()` - Only requires token for public IPs
  - Static serving at line 867-871
- `src-tauri/src/commands/webui.rs` - Dynamic control commands
- `src-tauri/tauri.conf.json` - Removed devUrl to fix desktop UI

### Frontend
- `src/components/settings/WebUiSettings.tsx` - Settings UI with updated warning text (line 220-230)
- `src/lib/commandClient.ts` - Tauri/browser abstraction
- `dist/` - Frontend build output

## Architecture

### Ports
- `15722` - WebUI API (auto-starts)
- `3000` - Vite dev server (dev only)

### Environment Variables
- `CC_SWITCH_WEBUI=0` - Disable auto-start
- `CC_SWITCH_WEBUI_HOST` - Bind address (default: 127.0.0.1)
- `CC_SWITCH_WEBUI_PORT` - Port (default: 15722)
- `CC_SWITCH_WEBUI_TOKEN` - Auth token (only for public IP)

### Authentication Logic (RFC 1918)
- **Private networks** (localhost, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, link-local): No token required
- **Public IPs**: Bearer token required
- Enforced at `require_auth` middleware (webui.rs:155-183)

## Decisions Made

1. **Auto-start by default**: WebUI starts unless `CC_SWITCH_WEBUI=0`
2. **RFC 1918 based auth**: Private networks免token, public IP需要token
3. **Separate PR/fork branches**: PR has sponsors, fork/main cleaned
4. **No devUrl in production**: Removed from tauri.conf.json to fix desktop UI loading
5. **UI color coding**: Blue info box for LAN mode (not amber warning)

## Git Branches

- `fork/main` (szh1118/cc-switch:main) - Clean README, all WebUI features
- `fork/feature/lan-webui` - PR branch with sponsors for upstream
- PR: https://github.com/farion1231/cc-switch/pull/3972

Both branches include all fixes and are up-to-date.

## Verification Status

### ✅ All Verified Working
1. **Desktop UI** - Loads correctly after removing devUrl
2. **WebUI localhost access** - `http://127.0.0.1:15722/` works without token
3. **WebUI LAN access** - `http://192.168.x.x:15722/` works without token
4. **Provider list** - Shows correctly in both desktop and browser
5. **Settings UI** - Correctly shows blue info "允许同一网络访问（RFC 1918，无需令牌）"

### Build Status
- Rust: Compiles successfully (release build ~4min)
- Frontend: `npx vite build` succeeds (~5.6s)
- Binary size: 31MB (with embedded frontend)

## Failed Attempts

1. **Initial loopback-only check** - Too restrictive, blocked LAN access
2. **Git author config** - Local git needs `GIT_AUTHOR_NAME/EMAIL` env vars

## Next Steps

None - all features complete and verified. Ready for upstream review.

## Open Questions

None
