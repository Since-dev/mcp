---
name: since-model-retirements
description: Use whenever code names an AI model ID (for example gpt-5, gpt-4o-mini-2024-07-18, claude-sonnet-4-5-20250929, gemini-2.5-pro, or a Bedrock, Vertex AI or Azure model ID), when choosing or switching a model, and when a model call starts failing with a model-not-found error. Asks Since.dev whether the model is deprecated or retiring, on which date on the platform the code calls, and what replaces it.
---

# Check AI model retirements with Since.dev

Providers retire models on fixed dates, and requests to a retired model fail. The date depends on where the code calls the model: the provider's own API, Azure, Amazon Bedrock and Google Vertex AI each set their own. Since.dev tracks every model's lifecycle on each of these platforms from the publishers' own pages.

## Check one model

```json
{ "model": "claude-sonnet-4-5-20250929" }
```

Call `check_code` with it. The answer gives the model's status (`supported`, `deprecated`, `retired` or `unknown`), every change with its date per platform, `daysLeft` and the `replacement`.

## Check every model in a file

Send the file to `check_code` as code, with its language or path:

```json
{ "code": "<the file's contents>", "language": "typescript" }
```

It finds each model ID in the code and reports the line it is on.

## Pick the right date

Look at how the code reaches the model. A client made for Azure, Bedrock or Vertex AI uses that platform's date, which can be earlier or later than the provider's own. Quote the date for the platform the code actually calls, not the first one you see.

## When a model is retiring

1. Move to the `replacement` the answer names.
2. Read the replacement's own changes with `whats_changed` (`{ "model": "<replacement id>" }`): a newer model can reject parameters the old one took.
3. Check again so nothing still names the old model.

## See what retires soon

```json
{ "provider": "openai", "within": "90d" }
```

Call `whats_changed` with a provider and `within` to list retirements in the coming days, soonest first.

`needs_review` and `unknown` mean not settled. Read the cited source or ask the user, never assume the model is fine.
