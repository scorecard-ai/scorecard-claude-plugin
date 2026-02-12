# Anthropic SDK Integration

Uses the proxy approach: redirect the Anthropic client's `baseURL` to Scorecard's proxy.

## JavaScript / TypeScript

1. Find **all** places where the Anthropic client is initialized (e.g., `new Anthropic(...)`)
2. Modify each to use the Scorecard proxy:

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

## Python

1. Find **all** places where the Anthropic client is initialized (e.g., `Anthropic(...)`)
2. Modify each to use the Scorecard proxy:

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

## Notes

- If the client has existing custom `defaultHeaders`, merge the Scorecard header into them
- Works with both sync and async clients (`Anthropic` and `AsyncAnthropic`)
- Streaming responses are fully supported through the proxy
