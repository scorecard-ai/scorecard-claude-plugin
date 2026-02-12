# LangChain Integration (Python)

Uses the OTEL approach via OpenLLMetry's Traceloop SDK.

## Setup

1. Install the required packages:

```bash
pip install traceloop-sdk opentelemetry-instrumentation-langchain
```

2. Add these env vars to `.env`:

```
TRACELOOP_API_KEY=<your_scorecard_api_key>
TRACELOOP_BASE_URL=https://tracing.scorecard.io/otel
SCORECARD_PROJECT_ID=<your-project-id>
```

3. Add Traceloop initialization **before** any LangChain imports. This is critical — import order matters:

```python
from traceloop.sdk import Traceloop
from traceloop.sdk.instruments import Instruments
import os

Traceloop.init(
    disable_batch=True,
    instruments={Instruments.LANGCHAIN},
    resource_attributes={
        "scorecard.project_id": os.getenv("SCORECARD_PROJECT_ID")
    }
)

# LangChain imports MUST come AFTER Traceloop.init()
from langchain_openai import ChatOpenAI
```

## What gets captured automatically

- LLM invocations with prompts and completions
- Chain execution paths with inputs/outputs
- Agent reasoning and tool selections
- Document retrieval operations
- Token consumption metrics
- Errors with full context

## Important notes

- **Import order is critical**: `Traceloop.init()` must be called before importing any LangChain modules, otherwise calls won't be instrumented
- `disable_batch=True` is recommended to ensure traces are sent immediately
- Console warnings about failed batch exports can be disregarded — traces transmit successfully
- Traces appear in the Records tab within 1-2 minutes
- The `TRACELOOP_API_KEY` should be set to your Scorecard API key (the same `ak_*` key)
