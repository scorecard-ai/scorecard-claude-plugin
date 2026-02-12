---
name: instrument
description: Set up Scorecard tracing for your LLM application. Use when the user wants to add observability, monitoring, or tracing to their AI/LLM app.
argument-hint: "[directory]"
disable-model-invocation: true
context: fork
allowed-tools: [Read, Glob, Grep, Edit, Write, Bash]
---

# Instrument with Scorecard

Set up Scorecard tracing for the user's LLM application.

## How Scorecard Tracing Works

Scorecard offers two instrumentation approaches:

- **Proxy approach** — Redirect the LLM client's `baseURL` to `https://llm.scorecard.io`. Scorecard transparently forwards requests to the LLM provider while capturing telemetry. Simplest setup (no extra dependencies). Works with OpenAI, Anthropic, and Azure OpenAI SDKs.
- **OTEL/SDK approach** — Send OpenTelemetry traces to `https://tracing.scorecard.io/otel`. Used by Claude Agent SDK, LangChain, and projects with existing OTEL instrumentation. Also available via the `scorecard-ai` wrapper SDK for deeper integration.

Both approaches support streaming responses.

## Project context (auto-detected)

- **Package dependencies:** !`cat package.json 2>/dev/null || cat requirements.txt 2>/dev/null || cat pyproject.toml 2>/dev/null || echo "No dependency file found"`
- **Existing env config:** !`cat .env 2>/dev/null || echo "No .env file found"`
- **OTEL presence:** !`grep -rl "opentelemetry\|TracerProvider\|OTLPTraceExporter\|trace.get_tracer\|OTEL_EXPORTER" --include="*.ts" --include="*.js" --include="*.py" --include="*.yaml" --include="*.yml" --include="*.env" . 2>/dev/null | head -5 || echo "None found"`
- **Scorecard presence:** !`grep -rl "scorecard\|llm\.scorecard\.io\|tracing\.scorecard\.io\|x-scorecard-api-key" --include="*.ts" --include="*.js" --include="*.py" --include="*.env" --include="*.json" --include="*.yaml" . 2>/dev/null | head -5 || echo "Not instrumented"`
- **Gitignore status:** !`grep -q "\.env" .gitignore 2>/dev/null && echo ".env is in .gitignore" || echo ".env is NOT in .gitignore (or .gitignore missing)"`

## Instructions

### Step 1: Check if Scorecard is already instrumented

Using the auto-detected "Scorecard presence" above, determine if tracing is already set up. If it is, tell the user tracing is already configured, show where, and stop. Do not make duplicate changes.

### Step 2: Detect the project's LLM setup

Using the auto-detected "Package dependencies" and further searching if needed, identify **all** LLM clients and frameworks in use. A project may use multiple — instrument all of them. Also detect the project language (JavaScript/TypeScript vs Python).

Check for each of these:

1. **Existing OpenTelemetry** — `@opentelemetry`, `opentelemetry-sdk`, `TracerProvider`, `NodeSDK`, `OTLPTraceExporter`, `trace.get_tracer`, or OTEL env vars
2. **Claude Agent SDK** — `@anthropic-ai/agent`, `anthropic-agent`, or `claude_agent_sdk`
3. **OpenAI SDK** — `openai` in dependencies, `from openai import OpenAI`, or `new OpenAI(`
4. **Anthropic SDK** — `anthropic` in dependencies, `from anthropic import Anthropic`, or `new Anthropic(`
5. **Azure OpenAI** — `@azure/openai`, `AzureOpenAI`, or `azure_endpoint`
6. **Vercel AI SDK** — `ai` or `@ai-sdk` in dependencies, imports from `ai` like `generateText`, `streamText`
7. **LangChain** — `langchain` in dependencies or imports

**If multiple integrations are found**, list all of them to the user before proceeding and instrument each one.

### Step 3: Implement instrumentation

Read the appropriate integration file(s) and apply the instructions:

- For existing OpenTelemetry setups, see [integrations/opentelemetry.md](integrations/opentelemetry.md)
- For OpenAI SDK (JS or Python), see [integrations/openai.md](integrations/openai.md)
- For Anthropic SDK (JS or Python), see [integrations/anthropic.md](integrations/anthropic.md)
- For Azure OpenAI, see [integrations/azure-openai.md](integrations/azure-openai.md)
- For Claude Agent SDK, see [integrations/claude-agent-sdk.md](integrations/claude-agent-sdk.md)
- For Vercel AI SDK, see [integrations/vercel-ai-sdk.md](integrations/vercel-ai-sdk.md)
- For LangChain, see [integrations/langchain.md](integrations/langchain.md)

When modifying client initializations, find **all** places where the client is initialized (not just the first one). If the client has existing custom options (e.g., custom `baseURL`, `defaultHeaders`, `timeout`), merge the Scorecard config with the existing options rather than replacing them.

### Step 4: Update .env and .gitignore

See [examples/env-template.md](examples/env-template.md) for all possible env vars by integration type.

After modifying code, ensure `.env` has the required variables:
- If `.env` doesn't exist, create it
- If it exists, append new variables (don't duplicate existing ones)

Check the auto-detected "Gitignore status" above. If `.env` is not in `.gitignore`:
- If `.gitignore` exists, add `.env` to it
- If `.gitignore` doesn't exist, create it with `.env` as the first entry

Remind the user to:
1. Get their API key from https://app.scorecard.io/settings/api-keys
2. Replace `<your_scorecard_api_key>` with their actual key (starts with `ak_`)

### Step 5: Verify and summarize

Summarize what was changed:
- Which file(s) were modified and what changed in each
- Which integration(s) were instrumented
- What env vars need to be set
- Any packages that need to be installed (remind user to run install commands)

Tell the user:
1. Set their `SCORECARD_API_KEY` in `.env`
2. Run their app and make a few LLM requests
3. Check https://app.scorecard.io — traces should appear in the Records tab within 1-2 minutes

### If no LLM usage found

If no LLM SDK, framework, or OpenTelemetry setup was detected, ask the user:
- What LLM provider or framework they are using
- Whether they have plans to add one

Point them to the docs for manual setup: https://docs.scorecard.io
