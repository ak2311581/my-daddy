# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

"Cheating Daddy" is an Electron desktop application that acts as a real-time AI assistant during video calls, interviews, and meetings. It captures screen and audio, then provides live contextual help via a transparent, always-on-top overlay window.

## Development Commands

```bash
npm install       # Install dependencies
npm start         # Launch the app in dev mode (Electron Forge)
npm run make      # Build distributable installers
npm run package   # Package without creating installers
npx prettier --write .  # Format code before committing
```

No test runner is configured yet. `npm run lint` is a no-op placeholder.

## Code Style

Prettier is configured (`.prettierrc`): 4-space indent, 150 print width, semicolons, single quotes. Run it before committing.

## Architecture

**Process model** — standard Electron two-process split:
- **Main process** (`src/index.js`): window management, IPC handlers, AI API calls, storage
- **Renderer process** (`src/index.html` + `src/components/`): UI built with Lit.js web components
- **Preload** (`src/preload.js`): exposes a restricted `window.api` bridge; all IPC must go through it

**UI layer** — currently Lit.js web components in `src/components/`. The plan is to migrate to React 19 + TypeScript + Tailwind + shadcn/ui (modeled after the `transcriber` sibling project). New UI code should use `src/components/ui/` and `@/` import prefix.

**AI providers** (`src/utils/`):
- `gemini.js` — Google Gemini 2.0 Flash Live (primary; handles audio/screen streaming)
- `cloud.js` — WebSocket cloud API (BYOK-free hosted option)
- `localai.js` — Local Ollama models

**Storage** (`src/storage.js`) — JSON files in the OS config dir:
- Windows: `%APPDATA%\cheating-daddy-config\`
- macOS: `~/Library/Application Support/cheating-daddy-config/`
- Linux: `~/.config/cheating-daddy-config/`

**Audio capture** — platform-specific:
- macOS: `src/assets/SystemAudioDump` binary
- Windows: Loopback device
- Linux: Microphone only

**Profiles & prompts** (`src/utils/prompts.js`) — profile-specific system prompts (Interview, Sales Call, Business Meeting, Presentation, Negotiation).

## IPC Pattern

All renderer↔main communication goes through named IPC channels. The preload script validates and exposes only whitelisted handlers. When adding a new feature that needs Node.js access, add a handler in `src/index.js` and expose it in `src/preload.js` — never use `nodeIntegration: true`.

## Future Migration Guidelines (from AGENTS.md)

When migrating or adding components toward the React/TS target:
- TypeScript strict mode — no `any`
- React functional components with hooks only
- shadcn/ui components in `src/components/ui/`
- Tailwind theming via CSS variables
- Maintain Electron context isolation throughout

## Upstream Sync

Cherry-pick selectively from the upstream `sohzm/cheating-daddy` repo. Verify locally after each merge before pushing.
