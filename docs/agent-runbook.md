# Hent-ai Agent Runbook

## Build

```bash
cd openclaw/
npm install        # if deps changed
```

No separate build step — TypeScript is loaded via tsx at runtime.

## Test

```bash
cd openclaw/
npx vitest run     # all tests, must pass before push
```

## Deploy

Plugin is loaded by OpenClaw gateway from `plugins.load.paths` config.
After code changes: gateway restart required (`openclaw gateway restart` from main session, NOT from Discord embedded session).

Current plugin path: `/Users/iyen/projects/Hent-ai/openclaw`

## Common Operations

### Switch channel mode
```bash
cd ~/projects/Hent-ai
npx tsx openclaw/scripts/set_channel_mode.ts --channel <ID> --mode private|default
```

### Check asset manifest
```bash
cat assets/manifest.json | jq .
```

### Check channel overrides
```bash
cat assets/channel-overrides.json | jq .
```

## Incident Patterns

### Cheer false positive (2026-05-19)
- Symptom: unwanted "화이팅!" + cheer.png sent to channel
- Cause: cheer intent classifier misclassified task request as emotional support
- Fix: tighten `buildCheerIntentPrompt` with negative examples
- Prevention: any prompt change → test with real frustration messages

### Manifest deletion (2026-05-18)
- Symptom: private asset set disappeared
- Cause: Python script deleted entire section instead of targeted edit
- Fix: manual manifest reconstruction
- Prevention: always `git diff` after any JSON manipulation script

### Path mismatch (2026-05-19)
- Symptom: private mode on but default images shown
- Cause: plugin imageDir pointed to old path, overrides saved to new path
- Fix: unified to ~/projects/Hent-ai as SSOT
- Prevention: after path changes, verify plugin's loaded imageDir in gateway logs

## Forbidden Actions

- `git push --force` on main
- Modifying production manifest.json without backup
- Running image generation in CI/test without mocks
- Merging PRs without test pass confirmation
