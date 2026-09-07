---
name: portless
description: Serve local development servers and static previews through stable, named HTTPS URLs (https://<name>.localhost) instead of raw http://localhost:<port>. Use whenever starting, running, previewing, or serving any local web app or dev server (Next.js, Vite, Astro, Nuxt, Express, React Router, Angular, Expo, React Native, Bun, HTML files, etc.), when a browser or agent needs a stable URL for a local web page, or when a dev server must be reachable from another machine (LAN, Tailscale, ngrok). Replaces random ports with clean, trusted HTTPS .localhost domains.
---

# Portless Local Development Workflow

Serve local dev servers and web previews through **`portless`** so they get stable, HTTPS-secured named URLs (`https://<name>.localhost`) instead of raw `http://localhost:<port>`.

## When to use

- Any local HTTP dev server or static preview that a browser, another service, or an agent needs to reach by URL.
- Multiple services at once (monorepos, API + web) where remembering ports is fragile.
- Anything that must keep a stable origin across restarts (OAuth redirects, cookies, CORS, `.env` base URLs).

## Requirements & Setup

- Node.js 24+, macOS / Linux / Windows.
- Install once: `npm install -g portless` (recommended) or per-project `npm install -D portless`.
- Verify with a read-only health check: `portless doctor` (proxy liveness, routes, DNS, CA trust). Run it first whenever routing or HTTPS looks wrong.
- First run auto-starts the proxy, generates a local CA, adds it to the OS trust store (current-user store on Windows / non-root macOS; system store when elevated), and binds port 443 (auto-elevates with sudo where the OS requires it). The proxy then reuses the last run's settings (port, TLS, TLDs) whenever it auto-starts.
- If a browser still shows certificate warnings, run `portless trust`. If the proxy is down, start it with `portless proxy start` (or just run an app — the proxy auto-starts).

## Core workflow

### Zero-config (any project with package.json)

```bash
portless                  # runs the "dev" script through the proxy
                          # -> https://<project>.localhost
portless --script start   # run a different script, e.g. "start"
```

The name is inferred from `package.json`, the git root, or the directory name. A `portless.json` (or a `"portless"` key in package.json) overrides it, e.g. `{ "name": "myapp" }` → `https://myapp.localhost`.

### Explicit name + command

```bash
portless <name> <command>
portless web vite         # -> https://web.localhost
portless api pnpm start   # -> https://api.localhost
portless docs.myapp next dev   # subdomains work too
```

`portless run <cmd>` infers the name; in a git worktree it prepends the branch name (`https://fix-ui.myapp.localhost`), so each worktree gets its own URL. Use `portless run --name <name> <cmd>` to override the base name while keeping the worktree prefix.

**Reserved words** cannot be used as app names directly: `run`, `get`, `alias`, `hosts`, `list`, `doctor`, `trust`, `clean`, `prune`, `proxy`, `service`. Use `portless run <command>` to infer a name, or `portless --name <name> <command>` to force one.

**Always read the real URL from `portless list` after launch** instead of guessing — names get transformed by package scopes, monorepo suffixes, and worktree prefixes.

### Static sites / plain HTML (no package.json)

```bash
portless <name> npx -y serve <directory>
portless demos npx -y serve .   # -> https://demos.localhost/<file>.html
```

`serve` honors the injected `PORT`, so no `-l` flag is needed.

### Monorepos

```bash
portless                  # from the repo root: starts every workspace package
                          # that has a "dev" script (Turborepo, pnpm/yarn/npm/bun)
cd apps/web && portless   # or start a single package
```

Default hostnames follow `<package>.<project>.localhost` (project name from the most common npm scope, else the workspace root directory name). An optional `portless.json` at the root overrides names via an `apps` map keyed by package path:

```json
{
  "apps": {
    "apps/web": { "name": "myapp" },
    "apps/api": { "name": "api.myapp" }
  }
}
```

### External services & Docker containers

```bash
portless alias <name> <port>       # static route to an already-running service
portless alias db 8080             # -> https://db.localhost
portless alias --remove <name>     # remove the route
```

## Framework notes

1. **Frameworks that ignore `PORT`** — Vite, VitePlus (`vp`), Astro, React Router, Angular (`ng`), Expo, React Native: portless auto-injects `--port <assigned>` into the server command. If the command already passes its own `--port`, portless respects it and does not add a duplicate. It also injects `--host 127.0.0.1` unless a host option is already present (Expo is the exception: `--host localhost`, and in LAN mode no `--host` at all). Vite / VitePlus / React Router additionally get `--strictPort`, so a busy port fails loudly instead of silently shifting. SvelteKit needs no entry: its dev server is Vite.
   - Injection only happens for server subcommands (`dev`, `serve`, `preview`, `start`, or a bare `vite`). Non-serving commands such as `vite build`, `vite optimize`, `astro check`, `vp test` are left untouched.
   - Next.js, Nuxt, and Express honor the injected `PORT`/`HOST` environment variables automatically.
   - Do NOT hardcode ports in config files. If a framework ends up listening on a different port than the one portless assigned, requests through the proxy return **502**.
2. **Scripts portless cannot rewrite** keep their own port: compound commands (`&&`, `|`, `;`), environment prefixes (`NODE_ENV=production vite`), delegation to another script (`npm run dev:vite`), runner flags before the script name (`bun run --bun`), or a trailing `#` comment. Set the port in the script yourself, otherwise the app returns 502 through the proxy.
3. **508 Loop Detected**: when a dev server proxies API calls to *another* portless app, rewrite the `Host` header with `changeOrigin: true`, otherwise portless routes the request back to the frontend in a loop:
   ```ts
   // vite.config.ts
   server: {
     proxy: {
       '/api': {
         target: 'https://api.myapp.localhost',
         changeOrigin: true,
         ws: true,
       },
     },
   }
   ```
   (webpack-dev-server: `changeOrigin: true` on the proxy rule.)
4. **LAN mode** (`portless proxy start --lan`, or `PORTLESS_LAN=1`): serves apps as `<name>.local` to devices on the same network via mDNS. For Next.js, add the `.local` hostnames to `allowedDevOrigins` in `next.config.js`. On Linux, LAN mode needs `avahi-utils` installed; macOS ships `dns-sd`.
5. **Sharing with teammates or the internet**: `--tailscale` / `--funnel` (needs the Tailscale CLI with HTTPS certs enabled) and `--ngrok` (needs the ngrok CLI, authenticated). `portless list` shows both local and shared URLs.
6. **Non-interactive environments** (`CI=1` or no TTY): portless exits with a descriptive error instead of prompting. Pre-start the proxy (`portless proxy start`) before running apps in task runners or CI.
7. **WebSockets & HMR** work through the proxy over HTTP/2 (and HTTP/1.1 `Upgrade`), so Vite/Next.js HMR needs no extra config.

## Environment variables

Injected into the child process:

| Variable | Meaning |
|---|---|
| `PORT` | Random free port (4000–4999) the app should listen on |
| `HOST` | Usually `127.0.0.1` |
| `PORTLESS_URL` | The public URL, e.g. `https://myapp.localhost` |
| `NODE_EXTRA_CA_CERTS` | Path to the portless CA (so Node trusts the proxy cert) |

Configuration (read by portless itself): `PORTLESS_PORT`, `PORTLESS_APP_PORT`, `PORTLESS_HTTPS=0` (same as `--no-tls`), `PORTLESS_TLD` (comma-separated list), `PORTLESS_LAN=1`, `PORTLESS_WILDCARD=1`, `PORTLESS_SYNC_HOSTS=0` (disable /etc/hosts auto-sync), `PORTLESS_STATE_DIR`, and `PORTLESS=0` to bypass portless and run the command directly.

## Verification & diagnostics

1. `portless list` — shows active routes and their real URLs (always the source of truth).
2. `portless doctor` — read-only health check (proxy, routes, DNS, CA trust); run this first.
3. `portless prune` — remove routes/processes orphaned by crashed sessions.
4. `portless trust` — add the local CA to the trust store (fixes browser warnings).
5. `portless hosts sync` (and `hosts clean`) — fix hostname resolution. `.localhost` resolves natively in Chrome/Firefox/Edge; Safari and custom TLDs (e.g. `.test`) may need the hosts entries.
6. `PORTLESS=0 <command>` — bypass the proxy for one command and run on the default port.
7. `portless clean` — full reset: proxy state, CA trust entry, hosts block (may require admin).

## Platform notes

- **Windows**: browsers resolve `.localhost` natively. If `curl` reports a certificate error against the local CA, Schannel needs CRL revocation checks disabled: `curl.exe -I --ssl-no-revoke https://<name>.localhost`.
- **macOS / Linux**: binding port 443 auto-elevates with sudo; running under sudo keeps state in the invoking user's `~/.portless`. On Linux, `portless trust` supports Debian/Ubuntu, Arch, Fedora/RHEL/CentOS, and openSUSE.
- **WSL**: portless updates both the Linux trust store and the Windows current-user Root store, so Windows browsers trust the certs.
- **Custom TLDs**: `portless proxy start --tld test` (repeat `--tld` for several) or a multi-segment TLD such as `dev.example.com` for production-parity URLs. Avoid `.local` (mDNS conflict) and `.dev` (HSTS). `PORTLESS_URL` uses the first configured TLD.
