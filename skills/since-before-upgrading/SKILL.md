---
name: since-before-upgrading
description: Use before upgrading, adding or removing a dependency, before editing a manifest or lockfile (package.json, package-lock.json, requirements.txt, pyproject.toml, poetry.lock, go.mod, Cargo.toml, Gemfile, pom.xml, build.gradle, composer.json and the like), and when a build breaks after a version bump. Asks Since.dev what the change breaks in this code, with the file and line, how severe it is, the days left and what to move to.
---

# Check a dependency change with Since.dev

Since.dev records every release, breaking change, deprecation and security advisory across 18 package registries, with its source. Use it so an upgrade lands working the first time.

## Before you change a version

1. **See what already affects the code.** Call `check_code` with the manifest or lockfile that pins the package:

   ```json
   { "manifest": { "path": "package.json", "content": "<the file's contents>" } }
   ```

   Use `lockfile` instead of `manifest` when the repo has one: it holds the exact installed versions, transitive ones included.

2. **Read what changed between your version and the one you want.** Call `whats_changed` with the package in any spelling or as a purl, and `since` set to the version you have now:

   ```json
   { "package": "npm:react", "since": "18.2.0" }
   ```

   Each change has its kind (removed, renamed, signature changed, deprecated, security), severity, dates, the replacement and a link to its source. `package.releasesBehind` says how far behind you are.

3. **Make the change, then check again** with the updated manifest or lockfile. Fix every `affected` finding before you finish.

## Reading the answer

- `affected`: the code uses the thing that changed. The finding names the file and line.
- `needs_review` and `unknown`: not settled. Read the cited source or ask the user. Never treat them as safe.
- `replacement`: what to move to, from the publisher's own notes.
- `daysLeft`: how long until a removal or end of life takes effect. Do the soonest first.

## When the user wants it fixed for them

If the repository is connected to Since.dev, `fix_code` with `repo` opens a pull request that fixes it, for a person to review. Your client asks before it runs. For a snippet, `fix_code` with `code` and `language` returns the patch.

## Cost

Each call costs credits from the workspace's monthly allowance. Checking a manifest or lockfile costs more than a single package, so check the file once, then read single packages as you need them.
