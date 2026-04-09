---
description: Safety-checked commit + push to origin/main. Usage: /publish "commit message"
---

Publish the current working tree to `origin/main` after running safety checks. The commit message is the argument to this command; if the user did not provide one, stop and ask for it.

## Safety checks (in order — abort on any failure)

1. **Working tree has changes.**
   ```bash
   git status --porcelain
   ```
   If empty → print "nothing to publish" and stop.

2. **CNAME integrity.** The file must exist and contain exactly `andrewuroskie.com` (with an optional trailing newline, nothing else).
   ```bash
   test -f CNAME || { echo "ABORT: CNAME missing"; exit 1; }
   content=$(tr -d '[:space:]' < CNAME)
   [ "$content" = "andrewuroskie.com" ] || { echo "ABORT: CNAME corrupted. Contents: $(cat CNAME)"; exit 1; }
   ```
   If this fails, **do not commit**. Tell the user exactly what CNAME contains and ask them to fix it (or offer to restore it to `andrewuroskie.com`).

3. **Large binary warning (non-blocking).** For any new or modified file >5MB, warn the user and list it. Proceed unless the user says stop.
   ```bash
   git status --porcelain | awk '{print $2}' | while read f; do
     [ -f "$f" ] || continue
     size=$(stat -f%z "$f" 2>/dev/null || stat -c%s "$f" 2>/dev/null)
     [ "$size" -gt 5242880 ] && echo "WARN: $f is $((size/1024/1024))MB"
   done
   ```

4. **Summary.** Print:
   - `git diff --stat` (tracked changes)
   - list of untracked files that will be added
   - the commit message

## Commit + push

```bash
git add -A
git commit -m "<message from user>"
git push origin main
```

Do **not** add Co-Authored-By trailers or any automation signature — this is a personal site and the user is the author.

## After push

1. Print the new commit hash (`git log --oneline -1`).
2. Print: "Live at https://andrewuroskie.com — GitHub Pages propagation typically takes 30–90 seconds."
3. Do not run any verification curl — propagation is asynchronous and the user can refresh the live preview tab or visit the URL directly.

## Rollback reminder

If the user wants to undo the publish immediately after, the safe move is:
```bash
git revert HEAD && git push origin main
```
Never `git reset --hard` a pushed commit. Never force-push.
