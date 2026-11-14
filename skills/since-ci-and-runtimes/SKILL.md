---
name: since-ci-and-runtimes
description: Use when editing CI workflows (.github/workflows/*.yml), Dockerfiles, docker-compose files or runtime version files (.nvmrc, .node-version, .python-version, .tool-versions, go.mod, global.json), when choosing a runtime or base image version, and when a CI runner or image fails. Asks Since.dev which runner images, base images and runtime versions are past or near end of life, and what to move to.
---

# Check runtimes, images and CI runners with Since.dev

Runtimes and CI runners reach end of life on published dates. After that, runtimes stop getting security fixes, and retired CI runner images make jobs fail. Since.dev tracks these dates from each project's own schedule.

## Runtime version files

Send the file to `check_code` as a manifest:

```json
{ "manifest": { "path": ".nvmrc", "content": "18\n" } }
```

This works for `.nvmrc`, `.node-version`, `.python-version`, `.tool-versions`, `go.mod` and `global.json`. The answer gives each runtime's status, end-of-life date and the line it is on.

## Dockerfiles, compose files and workflows

Send the file to `check_code` as code, with its path:

```json
{ "code": "<the file's contents>", "path": "Dockerfile" }
```

The same works for `docker-compose.yml` and `.github/workflows/ci.yml`. The answer names each base image's runtime (`FROM node:18-alpine` runs Node.js 18), each `runs-on:` runner label and each version a `setup-*` action installs, with its end-of-life date and the line it is on.

To check one version before you write it, give it as a surface:

```json
{ "surface": "runtime:nodejs@18" }
```

```json
{ "surface": "runtime:github-actions-runner-images@macos-14" }
```

The surface is `runtime:<product>@<version>`, for example `runtime:python@3.9`, `runtime:go@1.23` or `runtime:dotnet@8`.

## See what ends soon

```json
{ "surface": "runtime:nodejs", "within": "180d" }
```

With `whats_changed`, this lists the end-of-life dates coming up, soonest first.

## Moving off an old version

Prefer the newest supported LTS or the version the answer names as the replacement, change every place that pins it (the version file, the Dockerfile, the workflow), then check again. `needs_review` and `unknown` are not settled: read the cited source or ask the user.
