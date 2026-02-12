# Azure OpenAI Integration

Uses the proxy approach: redirect the Azure OpenAI client's `baseURL` to Scorecard's proxy.

## JavaScript / TypeScript

1. Find where the Azure OpenAI client is initialized (e.g., `new AzureOpenAI(...)`)
2. Modify it to use the Scorecard proxy:

```javascript
const client = new AzureOpenAI({
  apiKey: process.env.AZURE_OPENAI_API_KEY,
  baseURL: 'https://llm.scorecard.io',
  defaultHeaders: {
    'x-scorecard-api-key': process.env.SCORECARD_API_KEY,
  }
});
```

3. Add `SCORECARD_API_KEY` to `.env` file

## Python

1. Find where the Azure OpenAI client is initialized (e.g., `AzureOpenAI(...)`)
2. Modify it to use the Scorecard proxy:

```python
client = AzureOpenAI(
    api_key=os.environ.get("AZURE_OPENAI_API_KEY"),
    base_url="https://llm.scorecard.io",
    default_headers={
        "x-scorecard-api-key": os.environ.get("SCORECARD_API_KEY"),
    }
)
```

3. Add `SCORECARD_API_KEY` to `.env` file

## Notes

- The Scorecard proxy replaces the Azure endpoint — it forwards requests to the correct provider automatically
- If the client has existing custom `defaultHeaders`, merge the Scorecard header into them
