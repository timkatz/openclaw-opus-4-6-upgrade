# Upgrade OpenClaw to Claude Opus 4.6

Since OpenClaw hasn't officially added `claude-opus-4-6` to its model catalog yet, you need to patch it manually. This guide walks you through the process.

> ⚠️ **This patch lives in the container filesystem and will be lost on image rebuilds or container recreation.** Re-apply after updates until OpenClaw officially ships Opus 4.6 support.

## Steps

### 1. Find the model catalog file(s)

Inside your container:

```bash
find / -name "models.generated.js" -path "*pi-ai*" 2>/dev/null
```

You'll likely find two copies (one in the app's pnpm modules, one in the global npm install). **Patch both.**

### 2. Add the Opus 4.6 entries

In each file, search for `"claude-opus-4-5"`. There are two entries — one with `provider: "anthropic"`, one with `provider: "opencode"`.

Add the corresponding block **directly above** each one:

#### Anthropic provider entry:

```javascript
"claude-opus-4-6": {
  id: "claude-opus-4-6",
  name: "Claude Opus 4.6 (latest)",
  api: "anthropic-messages",
  provider: "anthropic",
  baseUrl: "https://api.anthropic.com",
  reasoning: true,
  input: ["text", "image"],
  cost: { input: 5, output: 25, cacheRead: 0.5, cacheWrite: 6.25 },
  contextWindow: 200000,
  maxTokens: 64000,
},
```

#### OpenCode provider entry:

```javascript
"claude-opus-4-6": {
  id: "claude-opus-4-6",
  name: "Claude Opus 4.6 (latest)",
  api: "anthropic-messages",
  provider: "opencode",
  baseUrl: "https://opencode.ai/zen",
  reasoning: true,
  input: ["text", "image"],
  cost: { input: 5, output: 25, cacheRead: 0.5, cacheWrite: 6.25 },
  contextWindow: 200000,
  maxTokens: 64000,
},
```

### 3. Update the config

```bash
openclaw config set agents.defaults.model.primary anthropic/claude-opus-4-6
```

### 4. Cold restart (required)

A hot reload will **NOT** pick up catalog changes. You must do a full restart:

```bash
# Docker:
docker restart <your-openclaw-container>

# Or if running natively:
openclaw gateway stop && openclaw gateway start
```

### 5. Verify

Check the logs for:

```
agent model: anthropic/claude-opus-4-6
```

If you see `Unknown model: anthropic/claude-opus-4-6`, the catalog patch didn't apply — double-check you edited the right file(s) and that both entries were added.

## Important Notes

- This patch lives in the filesystem and will be **lost** on image rebuilds or container recreation
- Re-apply after updates until OpenClaw officially ships Opus 4.6 support
- A **cold restart** is required — hot reload won't pick up catalog changes
- Make sure to patch **both** copies of `models.generated.js`

## License

MIT
