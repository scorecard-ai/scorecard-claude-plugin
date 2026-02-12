# Scorecard Plugin for Claude Code

Add LLM tracing to your AI application in seconds. This [Claude Code plugin](https://docs.anthropic.com/en/docs/claude-code/plugins) auto-detects your LLM SDKs and instruments them with [Scorecard](https://scorecard.io) — so you can monitor, evaluate, and debug your AI app from [app.scorecard.io](https://app.scorecard.io).

## Quick Start

```bash
claude --plugin-dir /path/to/scorecard-claude-plugin/scorecard-plugin
```

Then, from your project directory:

```
/scorecard:instrument
```

The plugin will detect your LLM setup, modify your code, and configure environment variables. That's it.

## What It Does

The `/scorecard:instrument` skill scans your project, identifies which LLM SDKs you're using, and applies the right instrumentation automatically. It handles:

- Adding Scorecard proxy configuration or OTEL exporters to your code
- Creating or updating your `.env` with the required variables
- Ensuring `.env` is in `.gitignore`
- Instrumenting **all** client initializations (not just the first one)
- Merging with existing custom headers and configuration

## Supported Integrations

| Integration | Language | Approach |
|-------------|----------|----------|
| OpenAI SDK | JS/TS, Python | Proxy — routes requests through `llm.scorecard.io` |
| Anthropic SDK | JS/TS, Python | Proxy |
| Azure OpenAI | JS/TS, Python | Proxy |
| Claude Agent SDK | JS/TS, Python | Env vars only — no code changes needed |
| Vercel AI SDK | JS/TS | SDK wrapper via `scorecard-ai` |
| LangChain | Python | OTEL via Traceloop SDK |
| Existing OpenTelemetry | JS/TS, Python | Adds Scorecard as an additional exporter |

## Prerequisites

1. A [Scorecard](https://scorecard.io) account
2. An API key from [app.scorecard.io/settings/api-keys](https://app.scorecard.io/settings/api-keys)
3. [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) CLI installed

## How It Works

Depending on your SDK, the plugin uses one of two approaches:

**Proxy** (OpenAI, Anthropic, Azure OpenAI) — Redirects your LLM client's base URL to `https://llm.scorecard.io`. Scorecard transparently forwards requests to the provider while capturing telemetry. No extra dependencies.

**OTEL/SDK** (Claude Agent SDK, Vercel AI SDK, LangChain, existing OTEL) — Sends OpenTelemetry traces to `https://tracing.scorecard.io/otel`. Used when the SDK has built-in tracing support or when wrapping with the `scorecard-ai` package.

Both approaches support streaming responses.

## After Instrumenting

1. Set your `SCORECARD_API_KEY` in `.env` (starts with `ak_`)
2. Run your app and make a few LLM requests
3. Check [app.scorecard.io](https://app.scorecard.io) — traces appear in the Records tab within 1–2 minutes

## Documentation

- [Scorecard Docs](https://docs.scorecard.io)
- [Claude Code Plugins](https://docs.anthropic.com/en/docs/claude-code/plugins)
