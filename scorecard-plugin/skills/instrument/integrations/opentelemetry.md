# OpenTelemetry Integration

If the project already has OpenTelemetry instrumentation, add Scorecard as an additional OTLP endpoint. The approach depends on how OTEL is configured.

## Option A: Environment variables

Add or update these env vars in `.env`:

```
OTEL_EXPORTER_OTLP_ENDPOINT=https://tracing.scorecard.io/otel
OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer <your_scorecard_api_key>"
```

If they already have an OTLP endpoint configured, they'll need to use a collector to fan out to multiple endpoints. Suggest setting up an OTEL Collector with multiple exporters (see Option B).

## Option B: OTEL Collector config

Add Scorecard as an exporter in their `otel-collector-config.yaml`:

```yaml
exporters:
  # ... existing exporters ...
  otlphttp/scorecard:
    endpoint: https://tracing.scorecard.io/otel
    headers:
      Authorization: "Bearer <your_scorecard_api_key>"

service:
  pipelines:
    traces:
      exporters: [<existing_exporters>, otlphttp/scorecard]
```

## Option C: OTEL in code (JavaScript)

Add a second exporter for Scorecard:

```javascript
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-http');

const scorecardExporter = new OTLPTraceExporter({
  url: 'https://tracing.scorecard.io/otel/v1/traces',
  headers: {
    'Authorization': `Bearer ${process.env.SCORECARD_API_KEY}`,
  },
});

// Add to your existing TracerProvider with a SimpleSpanProcessor or BatchSpanProcessor
tracerProvider.addSpanProcessor(new BatchSpanProcessor(scorecardExporter));
```

## Option D: OTEL in code (Python)

Add a second exporter for Scorecard:

```python
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter

scorecard_exporter = OTLPSpanExporter(
    endpoint="https://tracing.scorecard.io/otel/v1/traces",
    headers={"Authorization": f"Bearer {os.environ.get('SCORECARD_API_KEY')}"},
)

# Add to your existing TracerProvider
tracerProvider.add_span_processor(BatchSpanProcessor(scorecard_exporter))
```

After adding the Scorecard exporter, proceed to Step 4 in SKILL.md to set up the API key.
