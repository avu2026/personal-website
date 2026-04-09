# personal-website — Claude Code project brief

Personal site for Andrew V. Uroskie, served at **https://andrewuroskie.com**.

## Stack (deliberately minimal)

- Hand-coded static HTML. One file: `index.html` (~1,650 lines).
- Assets: `headshot.jpg`, `vdb_splash.webm`.
- **No framework, no build step, no bundler, no `package.json`.** Keep it that way.

## Deployment reality

- **GitHub Pages**, not Cloudflare. The site is served directly from `main` by GitHub.
- `CNAME` → `andrewuroskie.com`. DNS resolves to GH Pages A records `185.199.108–111.153`.
- There is no Cloudflare Pages project, no `wrangler.toml`, no GitHub Actions workflow. Do not add them.
- Propagation after `git push origin main` is typically **30–90 seconds**.

## File inventory

| Path | Role | Touch? |
|---|---|---|
| `index.html` | entire site | yes, main editing target |
| `CNAME` | custom domain binding | **never edit** — must always contain exactly `andrewuroskie.com` |
| `headshot.jpg` | portrait | replace only if asked |
| `vdb_splash.webm` | splash video | replace only if asked |
| `README.md` | repo description | rarely |
| `.claude/commands/` | slash commands for this project | yes |

## Workflow — edit → preview → publish

1. `/preview` — starts `browser-sync` in the background on `http://localhost:5500` and opens the browser. Live-reload on save.
2. Make edits to `index.html` (or assets).
3. `/diff` — inspect the working-tree diff before publishing.
4. `/publish "concise message"` — runs safety checks, commits, and pushes to `origin/main`. Live at `andrewuroskie.com` within ~60s.
5. Rollback any time: `git revert HEAD && git push`. History is never rewritten.

## Conventions

- **Direct-to-main.** No draft/preview branches.
- **Never rewrite history** (no force-push, no amend of pushed commits).
- **Keep the repo dependency-free.** Live-reload uses `npx --yes browser-sync` ephemerally; nothing is installed into the repo.
- Large binary additions (>5MB) get a warning at publish time. The site already ships `vdb_splash.webm`; think twice before adding more.
- If you edit `index.html`, preserve the existing structure and indentation style of surrounding code.

## Offline fallback

If npm is unreachable and `/preview` can't start `browser-sync`:
```
python3 -m http.server 8000
```
No live-reload, but the site renders at `http://localhost:8000`.
