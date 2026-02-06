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

## Context Window: 200K (not 1M)

You may have seen that Opus 4.6 supports a 1M token context window. **This is not available through Claude Code or Max plans.**

The 1M context window is an **API-only beta** that requires:

- **Usage tier 4** or custom rate limits on the direct API
- A **beta header**: `anthropic-beta: context-1m-2025-08-07` — it won't activate automatically
- **Premium pricing** past 200K tokens: 2x input ($10/MTok) and 1.5x output ($37.50/MTok)
- Available on: Direct API, Amazon Bedrock, Google Vertex AI, Microsoft Foundry

The standard context window for Opus 4.6 is **200K tokens**, which is what we set in the catalog entry (`contextWindow: 200000`). This is the correct value for Claude Code, Max subscriptions, and OpenClaw setups.

Even if your Anthropic API key is on tier 4, OpenClaw would need to pass the `anthropic-beta: context-1m-2025-08-07` header to use the 1M window — something that would require support in OpenClaw's API client code, not just the model catalog entry.

**TL;DR:** 1M context is API-only (beta, tier 4+, requires a special header, premium pricing above 200K). It's not automatically available through Claude Code or consumer plans as of today.

**Sources:**
- [Context Windows Documentation](https://platform.claude.com/docs/en/build-with-claude/context-windows)
- [Anthropic Pricing](https://platform.claude.com/docs/en/about-claude/pricing)
- [VentureBeat Coverage](https://venturebeat.com/technology/anthropics-claude-opus-4-6-brings-1m-token-context-and-agent-teams-to-take)
- [The Decoder Coverage](https://the-decoder.com/claude-opus-4-6-brings-one-million-token-context-window-to-anthropics-flagship-model/)

## Important Notes

- This patch lives in the filesystem and will be **lost** on image rebuilds or container recreation
- Re-apply after updates until OpenClaw officially ships Opus 4.6 support
- A **cold restart** is required — hot reload won't pick up catalog changes
- Make sure to patch **both** copies of `models.generated.js`

## License

MIT
