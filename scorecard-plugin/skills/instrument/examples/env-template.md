# Environment Variable Reference

All possible Scorecard-related env vars by integration type. Only add the vars relevant to the detected integration.

## Always required

```
SCORECARD_API_KEY=<your_scorecard_api_key>
```

Get this from https://app.scorecard.io/settings (starts with `ak_`).

## Proxy approach (OpenAI, Anthropic, Azure OpenAI)

No additional env vars beyond `SCORECARD_API_KEY`. The API key is passed via `x-scorecard-api-key` header in code.

## Claude Agent SDK

```
OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer <your_scorecard_api_key>"
ENABLE_BETA_TRACING_DETAILED=1
BETA_TRACING_ENDPOINT="https://tracing.scorecard.io/otel"
# Optional:
OTEL_RESOURCE_ATTRIBUTES="scorecard.project_id=<your-project-id>"
```

## Vercel AI SDK

```
SCORECARD_API_KEY=<your_scorecard_api_key>
# Optional:
SCORECARD_PROJECT_ID=<your-project-id>
```

## LangChain

```
TRACELOOP_API_KEY=<your_scorecard_api_key>
TRACELOOP_BASE_URL=https://tracing.scorecard.io/otel
SCORECARD_PROJECT_ID=<your-project-id>
```

## Existing OpenTelemetry

```
OTEL_EXPORTER_OTLP_ENDPOINT=https://tracing.scorecard.io/otel
OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer <your_scorecard_api_key>"
```
