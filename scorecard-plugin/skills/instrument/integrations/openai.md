# OpenAI SDK Integration

Uses the proxy approach: redirect the OpenAI client's `baseURL` to Scorecard's proxy.

## JavaScript / TypeScript

1. Find **all** places where the OpenAI client is initialized (e.g., `new OpenAI(...)`)
2. Modify each to use the Scorecard proxy:

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

## Python

1. Find **all** places where the OpenAI client is initialized (e.g., `OpenAI(...)` or `client = OpenAI()`)
2. Modify each to use the Scorecard proxy:

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

## Notes

- If the client has existing custom `defaultHeaders`, merge the Scorecard header into them
- If the client has an existing custom `baseURL`, the Scorecard proxy replaces it — the proxy forwards to the correct provider automatically
- Works with both sync and async clients (`OpenAI` and `AsyncOpenAI`)
- Streaming responses are fully supported through the proxy
