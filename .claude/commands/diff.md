---
description: Show the current working-tree diff so you can review changes before publishing
---

Show the user exactly what will be published.

Steps:

1. Run `git status --short` to list changed/untracked files.
2. Run `git diff --stat` for a file-by-file summary.
3. Run `git diff` for the full patch of tracked files.
4. If there are untracked files, list them separately — they will NOT be included by `/publish` unless explicitly staged first.
5. If the working tree is clean, say so plainly and stop.

Keep the output compact: summarize first, then include the full diff. Do not editorialize — the point is for the user to read the raw patch.
