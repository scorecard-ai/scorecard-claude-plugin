# Claude Agent SDK Integration

Uses the OTEL approach: the Claude Agent SDK has built-in tracing support via environment variables. No code changes required.

## Setup

Add these to the `.env` file (create if it doesn't exist):

```
OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer <your_scorecard_api_key>"
ENABLE_BETA_TRACING_DETAILED=1
BETA_TRACING_ENDPOINT="https://tracing.scorecard.io/otel"
```

Optionally, to target a specific Scorecard project:

```
OTEL_RESOURCE_ATTRIBUTES="scorecard.project_id=<your-project-id>"
```

## What gets captured automatically

- LLM API calls with full prompt/completion data
- Tool invocations with inputs and outputs
- Token counts (input, output, total)
- Model configuration and parameters
- Errors with full context

## Requirements

- Python v0.1.18+ or TypeScript v0.1.71+ of the Claude Agent SDK
- No code changes needed — the SDK reads these env vars automatically

## Notes

- If a project ID is not specified via `OTEL_RESOURCE_ATTRIBUTES`, traces default to the oldest project in the Scorecard account
- Traces appear in the Records tab within 1-2 minutes
