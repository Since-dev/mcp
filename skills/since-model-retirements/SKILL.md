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

An id written as a gateway's route (`vertex_ai/gemini-2.5-pro`, `openrouter/google/gemini-2.5-pro`) comes back `needs_review`: the provider's date is for its own API and may not apply where the gateway sends the request. Say whose date it is and check the gateway's own schedule.

## When a model is retiring

1. Move to the `replacement` the answer names. When the reason says requests keep working, answered by another model, nothing fails: the answers just come from that model, so tell the user instead of treating it as a break.
2. Read the finding's `replacementNotes` before you change the id. Each one is something the replacement does differently, from its own documentation: a default that changed, a setting it refuses or now needs, a difference in what comes back, with what keeps the old behavior. Make those changes together with the new id. A move is never the id alone when there are notes.
3. For anything the notes do not cover, read the replacement's own changes with `whats_changed` (`{ "model": "<replacement id>" }`).
4. Check again so nothing still names the old model.

## See what retires soon

```json
{ "provider": "openai", "within": "90d" }
```

Call `whats_changed` with a provider and `within` to list retirements in the coming days, soonest first.

`needs_review` and `unknown` mean not settled. Read the cited source or ask the user, never assume the model is fine.
