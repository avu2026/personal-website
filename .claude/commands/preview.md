---
description: Start a live-reload preview of the site at http://localhost:5500
---

Start a browser-sync live-reload preview of the current working tree and open it in the default browser.

Steps:

1. Check whether a preview is already running:
   ```bash
   if [ -f .browser-sync.pid ] && kill -0 "$(cat .browser-sync.pid)" 2>/dev/null; then
     echo "browser-sync already running (pid $(cat .browser-sync.pid)) at http://localhost:5500"
     open http://localhost:5500 2>/dev/null || true
     exit 0
   fi
   ```

2. Launch `browser-sync` via `npx` in the background (no install into the repo):
   ```bash
   nohup npx --yes browser-sync start \
     --server \
     --files "index.html,*.css,*.js,assets/**/*" \
     --port 5500 \
     --no-open \
     --no-notify \
     > .browser-sync.log 2>&1 &
   echo $! > .browser-sync.pid
   ```
   Use the Bash tool with `run_in_background: true` so the shell returns immediately.

3. Wait ~2 seconds, then verify the server is up:
   ```bash
   sleep 2 && curl -sI http://localhost:5500 | head -1
   ```
   Expect `HTTP/1.1 200 OK`.

4. Open the browser tab:
   ```bash
   open http://localhost:5500
   ```

5. Report status to the user: URL, pid, and the tail of `.browser-sync.log` if anything looked off.

**Fallback** — if `npx browser-sync` fails (e.g. offline), tell the user and offer:
```bash
python3 -m http.server 8000
```
No live-reload, but it serves the site at `http://localhost:8000`.

**To stop the preview** later:
```bash
kill "$(cat .browser-sync.pid)" && rm -f .browser-sync.pid
```
