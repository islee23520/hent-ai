# Hent-ai — AGENTS.md

OpenClaw emotion-image plugin. Classifies bot responses by emotion and attaches character images.

## Rules

- TypeScript: follow existing code style, strict mode.
- All changes must pass `npx vitest run` (232+ tests).
- Plugin is loaded by OpenClaw at runtime — changes require gateway restart or hot-reload.
- Direct Discord API calls (sendImageMessage, sendTextMessage) bypass gateway logs. Always add plugin-level logging.
- Image generation calls (cheer, onboarding) cost real money. Never trigger in tests without mocking.

## Build & Test

```bash
cd openclaw/
npx vitest run          # unit tests
npx tsx scripts/...     # utility scripts
```

## Forbidden

- Never commit API keys, tokens, or secrets.
- Never auto-merge PRs. Create PR only; merge is human-only.
- Never modify `manifest.json` without diff verification (2026-05-18 incident: script deleted entire private set).
- Never push to main without all tests passing.

## Key Paths

- `openclaw/index.ts` — main plugin entry
- `openclaw/assets/` — manifest, channel-overrides
- `assets/` — image files (sets/gothic-v1, sets/private)
- `openclaw/scripts/` — CLI utilities
- `openclaw/skills/` — OpenClaw skill definitions

## Intent Classifiers

- **Emotion classifier**: gpt-5.4-mini, classifies bot response → emotion (happy/neutral/loyalty/sorry/confused/focused)
- **Cheer intent**: gpt-5.4-mini, detects user asking for encouragement → generates image. HIGH false-positive risk — always add negative examples for task/debug requests.
- **Onboarding intent**: detects setup/configure requests

## Architecture Decisions

- Per-channel asset sets via channel-overrides.json
- Multi-character profiles planned (issue #70)
- Plugin sends images via Discord bot token directly, not through OpenClaw message pipeline
