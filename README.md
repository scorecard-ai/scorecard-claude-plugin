# Scorecard Claude Plugin

A Claude Code plugin for integrating with the [Scorecard](https://scorecard.io) AI evaluation platform.

## Features

- **LLM Tracing**: Instrument your AI applications with OpenTelemetry-based tracing
- **Evaluation**: Run evaluations and metrics on your LLM outputs
- **Observability**: Monitor AI application performance via Scorecard's dashboard

## Installation

Clone the repo and run Claude Code with the `--plugin-dir` flag:

```bash
git clone https://github.com/scorecard-ai/scorecard-claude-plugin.git
claude --plugin-dir /path/to/scorecard-claude-plugin/scorecard-plugin
```

## Commands

| Command | Description |
|---------|-------------|
| `/scorecard:instrument` | Set up Scorecard tracing for your LLM application |

## Supported Integrations

- OpenAI SDK (JavaScript, Python)
- Anthropic SDK (JavaScript, Python)
- Claude Agent SDK
- Vercel AI SDK
- Existing OpenTelemetry setups

## Configuration

1. Get your API key from [app.scorecard.io/settings/api-keys](https://app.scorecard.io/settings/api-keys)
2. Add to your `.env` file:
   ```
   SCORECARD_API_KEY=ak_your_api_key_here
   ```

## Documentation

For more information, visit [docs.scorecard.io](https://docs.scorecard.io)
