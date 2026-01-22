# Instrument with Scorecard

Set up Scorecard tracing for your LLM application.

## Instructions

### Step 1: Check for existing OpenTelemetry instrumentation

First, search the repo for existing OTEL setup:

1. **OTEL packages** - Look for `@opentelemetry`, `opentelemetry-sdk`, `opentelemetry-api` in dependencies
2. **OTEL config files** - Look for `otel-collector-config.yaml`, `otel-config.yaml`, or similar
3. **OTEL env vars** - Check `.env` files for `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_SERVICE_NAME`, etc.
4. **OTEL code** - Search for `TracerProvider`, `NodeSDK`, `OTLPTraceExporter`, or `trace.get_tracer`

#### If OTEL instrumentation exists

Add Scorecard as an additional OTLP endpoint. The approach depends on how OTEL is configured:

**Option A: If using environment variables**

Add or update these env vars in `.env`:

```
OTEL_EXPORTER_OTLP_ENDPOINT=https://tracing.scorecard.io/otel
OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer <your_scorecard_api_key>"
```

If they already have an OTLP endpoint configured, they'll need to use a collector to fan out to multiple endpoints. Suggest setting up an OTEL Collector with multiple exporters.

**Option B: If using OTEL Collector config**

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

**Option C: If configuring OTEL in code (JavaScript)**

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

**Option D: If configuring OTEL in code (Python)**

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

After adding the Scorecard exporter, skip to Step 3 to set up the API key.

---

### Step 2: Detect the LLM setup (if no OTEL found)

If no existing OTEL instrumentation was found, search the repo for how the user is calling LLMs:

1. **Claude Agent SDK** - Look for `@anthropic-ai/agent` or `anthropic-agent`
2. **OpenAI SDK** - Look for `openai` in dependencies or imports like `from openai import OpenAI`
3. **Anthropic SDK** - Look for `anthropic` in dependencies or imports like `from anthropic import Anthropic`
4. **Vercel AI SDK** - Look for `ai` or `@ai-sdk` in dependencies

### Step 2: Implement instrumentation based on what's found

---

#### If using Claude Agent SDK

Add these to the `.env` file (create if it doesn't exist):

```
OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer <your_scorecard_api_key>"
ENABLE_BETA_TRACING_DETAILED=1
BETA_TRACING_ENDPOINT="https://tracing.scorecard.io/otel"
```

Tell the user to replace `<your_scorecard_api_key>` with their API key from https://app.scorecard.io/settings/api-keys

---

#### If using OpenAI SDK (JavaScript)

1. Find where the OpenAI client is initialized (e.g., `new OpenAI(...)`)
2. Modify it to use the Scorecard proxy:

```javascript
const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
  baseURL: 'https://llm.scorecard.io',
  defaultHeaders: {
    'x-scorecard-api-key': process.env.SCORECARD_API_KEY,
  }
});
```

3. Add `SCORECARD_API_KEY` to `.env` file

---

#### If using OpenAI SDK (Python)

1. Find where the OpenAI client is initialized (e.g., `OpenAI(...)` or `client = OpenAI()`)
2. Modify it to use the Scorecard proxy:

```python
client = OpenAI(
    api_key=os.environ.get("OPENAI_API_KEY"),
    base_url="https://llm.scorecard.io",
    default_headers={
        "x-scorecard-api-key": os.environ.get("SCORECARD_API_KEY"),
    }
)
```

3. Add `SCORECARD_API_KEY` to `.env` file

---

#### If using Anthropic SDK (JavaScript)

1. Find where the Anthropic client is initialized (e.g., `new Anthropic(...)`)
2. Modify it to use the Scorecard proxy:

```javascript
const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
  baseURL: 'https://llm.scorecard.io',
  defaultHeaders: {
    'x-scorecard-api-key': process.env.SCORECARD_API_KEY,
  }
});
```

3. Add `SCORECARD_API_KEY` to `.env` file

---

#### If using Anthropic SDK (Python)

1. Find where the Anthropic client is initialized (e.g., `Anthropic(...)`)
2. Modify it to use the Scorecard proxy:

```python
client = Anthropic(
    api_key=os.environ.get("ANTHROPIC_API_KEY"),
    base_url="https://llm.scorecard.io",
    default_headers={
        "x-scorecard-api-key": os.environ.get("SCORECARD_API_KEY"),
    }
)
```

3. Add `SCORECARD_API_KEY` to `.env` file

---

#### If using Vercel AI SDK

1. Install the Scorecard wrapper: suggest running `npm install scorecard-ai`
2. Find where AI SDK is used and wrap the model provider
3. Link to docs for details: https://docs.scorecard.io/tracing

---

### Step 3: Update .env file

After modifying the code, ensure the `.env` file has:

```
SCORECARD_API_KEY=<your_scorecard_api_key>
```

If `.env` doesn't exist, create it. If it exists, append the new variable.

Remind the user to:
1. Get their API key from https://app.scorecard.io/settings/api-keys
2. Replace `<your_scorecard_api_key>` with their actual key (starts with `ak_`)
3. Make sure `.env` is in `.gitignore`

---

### Step 4: Confirm changes

Summarize what was changed:
- Which file(s) were modified
- What the client initialization looks like now
- What env vars need to be set

Tell user to run their app and check https://app.scorecard.io to see traces.

---

### If no LLM usage found

Ask the user what LLM provider they're using, or point them to docs: https://docs.scorecard.io/tracing
