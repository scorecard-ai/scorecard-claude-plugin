# Manual OTEL Instrumentation (fallback)

The recommended path for apps **not** built on the Claude Agent SDK (raw OpenAI/Anthropic/LangChain/Vercel/custom). Emit OpenTelemetry spans in the **same shape the Claude Agent SDK emits**, so Scorecard ingests and renders them identically — the full turn-by-turn conversation view — but branded with the app's own `service.name`.

## Why the span names stay `claude_code.*`

Scorecard's trace parser keys the rich conversation view off three literal span names (`claude_code.interaction`, `claude_code.llm_request`, `claude_code.tool`) and the tracer scope `com.anthropic.claude_code.tracing`. These are an internal contract — **keep them exactly as written**. The user never sees them; the UI shows `service.name`, which you set to the app's name. Renaming the spans silently breaks the turn view.

## Setup

### JavaScript / TypeScript

1. Install dependencies:

```bash
npm install @opentelemetry/api @opentelemetry/sdk-trace-node @opentelemetry/sdk-trace-base @opentelemetry/exporter-trace-otlp-http @opentelemetry/resources
```

2. Create `scorecardTracing.ts`:

```typescript
import { context, trace } from '@opentelemetry/api';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { resourceFromAttributes } from '@opentelemetry/resources';
import { BatchSpanProcessor } from '@opentelemetry/sdk-trace-base';
import { NodeTracerProvider } from '@opentelemetry/sdk-trace-node';

const provider = new NodeTracerProvider({
  resource: resourceFromAttributes({
    'service.name': process.env.SCORECARD_SERVICE_NAME ?? 'my-agent',
    'scorecard.project_id': process.env.SCORECARD_PROJECT_ID ?? '',
  }),
  spanProcessors: [
    new BatchSpanProcessor(
      new OTLPTraceExporter({
        url: 'https://tracing.scorecard.io/otel/v1/traces',
        headers: { Authorization: `Bearer ${process.env.SCORECARD_API_KEY}` },
      }),
    ),
  ],
});
provider.register();

// Keep this scope name — Scorecard's Claude Code adapter keys off it.
const tracer = trace.getTracer('com.anthropic.claude_code.tracing', '1.0.0');

/** Root span for one user turn. Runs `fn` with the interaction span active. */
export async function traceInteraction<T>(
  opts: { userPrompt: string; sessionId: string },
  fn: () => Promise<T>,
): Promise<T> {
  const span = tracer.startSpan('claude_code.interaction', {
    attributes: {
      'span.type': 'interaction',
      user_prompt: opts.userPrompt,
      'session.id': opts.sessionId,
    },
  });
  try {
    return await context.with(trace.setSpan(context.active(), span), fn);
  } finally {
    span.end();
  }
}

/** One LLM call. `newContext` is the conversation delta (see "Envelope" below). */
export async function traceLLMRequest(
  opts: { model: string; newContext: string; sessionId: string },
  run: () => Promise<{ output: string; inputTokens: number; outputTokens: number }>,
): Promise<string> {
  return tracer.startActiveSpan('claude_code.llm_request', async (span) => {
    span.setAttributes({
      'span.type': 'llm_request',
      model: opts.model,
      new_context: opts.newContext,
      'session.id': opts.sessionId,
    });
    try {
      const r = await run();
      span.setAttributes({
        input_tokens: r.inputTokens,
        output_tokens: r.outputTokens,
        'response.model_output': r.output,
      });
      return r.output;
    } finally {
      span.end();
    }
  });
}

/** One tool invocation. */
export async function traceToolCall<T>(
  opts: { name: string; input: unknown },
  run: () => Promise<T>,
): Promise<T> {
  return tracer.startActiveSpan('claude_code.tool', async (span) => {
    span.setAttributes({
      tool_name: opts.name,
      tool_input: `[TOOL INPUT: ${opts.name}]\n${JSON.stringify(opts.input)}`,
    });
    try {
      const result = await run();
      span.setAttribute('new_context', `[TOOL RESULT: ${opts.name}]\n${String(result)}`);
      return result;
    } finally {
      span.end();
    }
  });
}
```

3. Wrap the LLM calls:

```typescript
import OpenAI from 'openai';
import { traceInteraction, traceLLMRequest } from './scorecardTracing';

const client = new OpenAI();

async function handleUserMessage(userMessage: string, sessionId: string) {
  return traceInteraction({ userPrompt: userMessage, sessionId }, async () => {
    return traceLLMRequest(
      { model: 'gpt-4o', newContext: `[USER PROMPT]\n${userMessage}`, sessionId },
      async () => {
        const res = await client.chat.completions.create({
          model: 'gpt-4o',
          messages: [{ role: 'user', content: userMessage }],
        });
        return {
          output: res.choices[0].message.content ?? '',
          inputTokens: res.usage?.prompt_tokens ?? 0,
          outputTokens: res.usage?.completion_tokens ?? 0,
        };
      },
    );
  });
}
```

### Python

1. Install dependencies:

```bash
pip install opentelemetry-sdk opentelemetry-exporter-otlp-proto-http
```

2. Create `scorecard_tracing.py`:

```python
import json
import os
from contextlib import contextmanager

from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.resources import Resource
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

_provider = TracerProvider(
    resource=Resource.create(
        {
            "service.name": os.environ.get("SCORECARD_SERVICE_NAME", "my-agent"),
            "scorecard.project_id": os.environ.get("SCORECARD_PROJECT_ID", ""),
        }
    )
)
_provider.add_span_processor(
    BatchSpanProcessor(
        OTLPSpanExporter(
            endpoint="https://tracing.scorecard.io/otel/v1/traces",
            headers={"Authorization": f"Bearer {os.environ['SCORECARD_API_KEY']}"},
        )
    )
)
trace.set_tracer_provider(_provider)

# Keep this scope name — Scorecard's Claude Code adapter keys off it.
_tracer = trace.get_tracer("com.anthropic.claude_code.tracing", "1.0.0")


@contextmanager
def trace_interaction(user_prompt: str, session_id: str):
    with _tracer.start_as_current_span("claude_code.interaction") as span:
        span.set_attribute("span.type", "interaction")
        span.set_attribute("user_prompt", user_prompt)
        span.set_attribute("session.id", session_id)
        yield span


@contextmanager
def trace_llm_request(model: str, new_context: str, session_id: str):
    """Yields a dict — set output / input_tokens / output_tokens after the call."""
    result: dict = {}
    with _tracer.start_as_current_span("claude_code.llm_request") as span:
        span.set_attribute("span.type", "llm_request")
        span.set_attribute("model", model)
        span.set_attribute("new_context", new_context)
        span.set_attribute("session.id", session_id)
        yield result
        span.set_attribute("input_tokens", result.get("input_tokens", 0))
        span.set_attribute("output_tokens", result.get("output_tokens", 0))
        span.set_attribute("response.model_output", result.get("output", ""))


@contextmanager
def trace_tool_call(name: str, tool_input):
    with _tracer.start_as_current_span("claude_code.tool") as span:
        span.set_attribute("tool_name", name)
        span.set_attribute("tool_input", f"[TOOL INPUT: {name}]\n{json.dumps(tool_input)}")
        result: dict = {}
        yield result
        span.set_attribute("new_context", f"[TOOL RESULT: {name}]\n{result.get('output', '')}")
```

3. Wrap the LLM calls:

```python
from openai import OpenAI
from scorecard_tracing import trace_interaction, trace_llm_request

client = OpenAI()


def handle_user_message(user_message: str, session_id: str) -> str:
    with trace_interaction(user_message, session_id):
        with trace_llm_request("gpt-4o", f"[USER PROMPT]\n{user_message}", session_id) as out:
            res = client.chat.completions.create(
                model="gpt-4o",
                messages=[{"role": "user", "content": user_message}],
            )
            out["output"] = res.choices[0].message.content or ""
            out["input_tokens"] = res.usage.prompt_tokens
            out["output_tokens"] = res.usage.completion_tokens
        return out["output"]
```

## The `new_context` envelope

`new_context` on the LLM span is the conversation delta, built from labeled blocks. Match this format exactly — the parser splits on `\n---\n` between blocks:

- **First turn:** `[USER PROMPT]\n<the user's message>`
- **Follow-up turns (after tool calls):** one block per tool result, joined by `\n---\n`:
  ```
  [TOOL RESULT: <tool_use_id>]\n<result text>
  ---
  [TOOL RESULT: <tool_use_id>]\n<result text>
  ```

Other attributes carry the rest:
- **Authoritative user prompt** → `user_prompt` on the interaction span (not `new_context`).
- **Assistant reply** → `response.model_output` on the LLM span.
- **Tool call** → `tool_name`, `tool_input` = `[TOOL INPUT: <name>]\n<json args>`, and its result in `new_context` = `[TOOL RESULT: <name>]\n<result>`.
- **Tokens** → `input_tokens` / `output_tokens` (integers). Only these two are counted; cache tokens are cosmetic.

## Linking and routing

- **Group turns into one record:** use the same `session.id` across every span in a conversation. Scorecard combines traces that share a `session.id` into a single record.
- **Route to a project:** `scorecard.project_id` on resource attributes (set here via `SCORECARD_PROJECT_ID`). Missing/invalid → traces land in your org's oldest project.
- **Auth / org:** the `Authorization: Bearer <ak_...>` header is validated by Scorecard's collector, which stamps your org onto the trace. Without a valid key the trace is dropped.

## Env vars

```
SCORECARD_API_KEY=<your_scorecard_api_key>
SCORECARD_PROJECT_ID=<your-project-id>
SCORECARD_SERVICE_NAME=<your-app-name>
```
