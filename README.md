# Scorecard Plugin for Claude Code

Add LLM tracing to your AI application in seconds. This [Claude Code plugin](https://docs.anthropic.com/en/docs/claude-code/plugins) auto-detects your LLM SDKs and instruments them with [Scorecard](https://scorecard.io) — so you can monitor, evaluate, and debug your AI app from [app.scorecard.io](https://app.scorecard.io).

It also bundles the [Scorecard MCP server](#scorecard-mcp-server), giving Claude Code direct access to your Scorecard workspace — projects, testsets, metrics, runs, and more — without leaving your editor.

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

Approaches are applied in priority order — built-in OTEL first, manual OTEL next, proxy/wrapper last.

| Integration | Language | Approach |
|-------------|----------|----------|
| Claude Agent SDK | JS/TS, Python | **Built-in OTEL** — env vars only, no code changes |
| Existing OpenTelemetry | JS/TS, Python | **OTEL** — adds Scorecard as an additional exporter |
| LangChain | Python | **OTEL** — via Traceloop / OpenLLMetry |
| Any other app (raw OpenAI/Anthropic/custom) | JS/TS, Python | **Manual OTEL** — emits Agent-SDK-shaped spans |
| OpenAI SDK | JS/TS, Python | Proxy _(lower-priority)_ — routes through `llm.scorecard.io` |
| Anthropic SDK | JS/TS, Python | Proxy _(lower-priority)_ |
| Azure OpenAI | JS/TS, Python | Proxy _(lower-priority)_ |
| Vercel AI SDK | JS/TS | SDK wrapper _(lower-priority)_ — via `scorecard-ai` |

## Scorecard MCP Server

Installing the plugin also connects the [Scorecard MCP server](https://mcp.scorecard.io/mcp), which lets Claude Code read and manage your Scorecard workspace directly — ask for what you want in plain language and Claude calls the tools on your behalf.

Available tools cover:

- **Projects** — list and create
- **Testsets & testcases** — create, read, update, delete
- **Metrics** — create, read, update, delete evaluation metrics
- **Runs & records** — list runs, list/create records, upsert scores
- **Systems** — manage system configurations and versions
- **Docs** — search the Scorecard documentation

For example: *"Create a testset in my project from these examples"* or *"List my last 5 runs and show their scores."*

Claude Code connects to the server automatically once the plugin is loaded. Run `/mcp` to check the connection and complete any authentication it prompts for.

## Prerequisites

1. A [Scorecard](https://scorecard.io) account
2. An API key from [app.scorecard.io/settings](https://app.scorecard.io/settings)
3. [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) CLI installed

## How It Works

Scorecard ingests OpenTelemetry traces. The plugin applies the highest-priority approach that fits your stack:

1. **Built-in OTEL via env vars (preferred)** — For the Claude Agent SDK (and other frameworks with built-in OTEL), tracing is enabled purely with environment variables — no code changes. Traces go to `https://tracing.scorecard.io/otel`.
2. **OTEL instrumentation (fallback)** — For any other app, the plugin emits OpenTelemetry traces to Scorecard. Existing-OTEL and LangChain apps use their standard exporters; everything else is instrumented to emit spans in the **same shape the Claude Agent SDK emits**, so they render identically in Scorecard — branded with your app's own service name.
3. **Proxy / SDK wrapper (lower-priority)** — Simpler but less faithful. Redirects your client's base URL to `https://llm.scorecard.io` (proxy) or wraps the Vercel AI SDK with `scorecard-ai`. Used only if you prefer a minimal-code option.

All approaches support streaming responses.

## After Instrumenting

1. Set your `SCORECARD_API_KEY` in `.env` (starts with `ak_`)
2. Run your app and make a few LLM requests
3. Check [app.scorecard.io](https://app.scorecard.io) — traces appear in the Records tab within 1–2 minutes

## Documentation

- [Scorecard Docs](https://docs.scorecard.io)
- [Claude Code Plugins](https://docs.anthropic.com/en/docs/claude-code/plugins)
