# Scorecard Claude Plugin

A Claude Code plugin for integrating with the [Scorecard](https://scorecard.io) AI evaluation platform.

## Features

- **LLM Tracing**: Instrument your AI applications with OpenTelemetry-based tracing
- **Evaluation**: Run evaluations and metrics on your LLM outputs
- **Observability**: Monitor AI application performance via Scorecard's dashboard

## Installation

Add this plugin to your Claude Code configuration.

## Commands

- `/instrument` - Set up Scorecard tracing for your LLM application (supports OpenAI, Anthropic, Vercel AI SDK, and Claude Agent SDK)

## Configuration

The plugin connects to Scorecard's MCP server at `https://mcp.scorecard.io/mcp`.

You'll need a Scorecard API key from https://app.scorecard.io/settings/api-keys

## Documentation

For more information, visit https://docs.scorecard.io
