---
name: since-what-changed
description: Use before writing code against a library, SDK, framework, API or runtime you learned about in training, especially fast-moving ones, and when the user asks what changed in a package, provider, model or API since a date or version. Asks Since.dev what changed after your training cutoff, so the code matches the versions that exist today.
---

# Ask Since.dev what changed since you were trained

You learned these packages and APIs at a point in time. Since then, functions were renamed or removed, parameters changed, models retired and runtimes reached end of life. Since.dev keeps the record of every change, with its source, so you can catch up before you write code.

## Catch up on one thing

Call `whats_changed` with what you are about to use and `since` set to your training cutoff as a date:

```json
{ "package": "pypi:openai", "since": "2025-01-01" }
```

Instead of `package`, use `provider` (for example `stripe` or `anthropic`), `model`, or `surface` (an API host such as `api.stripe.com`, or a runtime such as `runtime:node@20`). Use a version instead of a date when you know which one the project has: `"since": "1.40.0"`.

Each change gives its kind, severity, dates, the replacement and a link to the publisher's source. Page through more with `nextCursor`. To read one change in full, call `whats_changed` with its `id`.

## Check the code you wrote

After writing it, call `check_code` with the snippet and its language. It reports anything in the code that a recorded change affects, with the line.

## Ask whether something is still supported

```json
{ "surface": "runtime:python@3.9" }
```

With `check_code`, the answer says `supported`, `deprecated`, `retired` or `unknown`, with its source.

Treat `needs_review` and `unknown` as not settled. When an answer cites a source, prefer it over what you remember.
