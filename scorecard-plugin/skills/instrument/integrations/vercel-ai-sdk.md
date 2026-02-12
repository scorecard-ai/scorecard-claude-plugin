# Vercel AI SDK Integration

Uses the SDK wrapper approach: wrap the `ai` module with Scorecard's `wrapAISDK` function.

## Setup

1. Install the Scorecard wrapper:

```bash
npm install scorecard-ai
```

2. Find where the AI SDK is used (look for imports from `ai` like `generateText`, `streamText`, `generateObject`, `streamObject`).

3. Wrap the `ai` module with Scorecard:

```typescript
import { openai } from '@ai-sdk/openai';
import * as ai from 'ai';
import { wrapAISDK } from 'scorecard-ai';

const aiSDK = wrapAISDK(ai);

// Replace direct ai.generateText / ai.streamText calls with aiSDK.generateText / aiSDK.streamText
const { text } = await aiSDK.generateText({
  model: openai('gpt-4o-mini'),
  prompt: 'What is the capital of France?',
});
```

4. Add `SCORECARD_API_KEY` to `.env` file

## Automatically instrumented functions

The wrapper instruments these functions — all other functions pass through unchanged:
- `generateText`
- `generateObject`
- `streamText`
- `streamObject`
- `embed`
- `embedMany`

## Optional: Automatic metrics

Configure automatic scoring with metrics:

```typescript
const aiSDK = wrapAISDK(ai, {
  projectId: process.env.SCORECARD_PROJECT_ID,
  metrics: [
    'metric-id-123',
    'Check if the response is helpful and accurate',
  ],
});
```

Scorecard auto-creates metrics from natural language descriptions.

## Configuration options

| Parameter            | Type     | Default              | Purpose                                    |
| -------------------- | -------- | -------------------- | ------------------------------------------ |
| `projectId`          | string   | `SCORECARD_PROJECT_ID` env | Links traces to a project (required for metrics) |
| `metrics`            | string[] | `[]`                 | Metric IDs or natural language descriptions |
| `apiKey`             | string   | `SCORECARD_API_KEY` env | Authentication                             |
| `serviceName`        | string   | `"ai-sdk-app"`       | Telemetry identifier                       |
| `serviceVersion`     | string   | undefined            | Version tracking across releases           |
| `maxExportBatchSize` | number   | 1                    | Batch traces before export                 |

## Notes

- The wrapper works with any AI SDK model provider (OpenAI, Anthropic, Google, etc.)
- Traces transmit automatically — no additional code needed beyond the wrap call
- The `SCORECARD_API_KEY` env var is read automatically if not passed in options
