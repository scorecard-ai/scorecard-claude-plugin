# Environment Variable Reference

All possible Scorecard-related env vars by integration type. Only add the vars relevant to the detected integration.

## Always required

Every integration needs a Scorecard API key — get it from https://app.scorecard.io/settings (starts with `ak_`). The env var it travels under depends on the integration:

- **Proxy (OpenAI, Anthropic, Azure OpenAI) and Vercel AI SDK** → `SCORECARD_API_KEY`
- **Claude Agent SDK** → carried inside `OTEL_EXPORTER_OTLP_HEADERS` as `Authorization=Bearer <key>`
- **LangChain** → `TRACELOOP_API_KEY`

```
SCORECARD_API_KEY=<your_scorecard_api_key>
```

## Proxy approach (OpenAI, Anthropic, Azure OpenAI)

No additional env vars beyond `SCORECARD_API_KEY`. The API key is passed via `x-scorecard-api-key` header in code.

## Claude Agent SDK

```
OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer <your_scorecard_api_key>"
ENABLE_BETA_TRACING_DETAILED=1
BETA_TRACING_ENDPOINT="https://tracing.scorecard.io/otel"
OTEL_LOG_USER_PROMPTS=1
OTEL_LOG_TOOL_DETAILS=1
OTEL_LOG_TOOL_CONTENT=1
# Optional:
OTEL_RESOURCE_ATTRIBUTES="scorecard.project_id=<your-project-id>"
```

The three `OTEL_LOG_*` vars capture prompt and tool content — without `OTEL_LOG_USER_PROMPTS=1`, the user prompt is recorded as `<REDACTED>`.

## Manual OTEL instrumentation (fallback)

```
SCORECARD_API_KEY=<your_scorecard_api_key>
SCORECARD_PROJECT_ID=<your-project-id>
SCORECARD_SERVICE_NAME=<your-app-name>
```

The exporter sends to `https://tracing.scorecard.io/otel/v1/traces` with `Authorization: Bearer $SCORECARD_API_KEY`. `SCORECARD_SERVICE_NAME` is the name shown in the Scorecard UI.

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
