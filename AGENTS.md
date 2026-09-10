# Excalidraw — Base44 dev environment

## Stack
Yarn workspaces monorepo (Vite 5 + React 19). The user-facing app is `excalidraw-app/`;
the editor core lives in `packages/excalidraw`. Vite resolves workspace packages via
source aliases (no pre-build step needed for dev).

## Running here
`docker compose -f docker-compose.base44.yml up -d` — runs `node:24`, bind-mounts the
repo, installs workspace deps with `yarn --frozen-lockfile`, then runs
`yarn vite --host --port 3000 --no-open` from `excalidraw-app/`.

Dev server is on host port **3000**.

## Quirks (non-obvious)
- `.env.development` sets `VITE_APP_PORT=3001`; the compose overrides it to `3000` via
  the `environment:` block so the preview proxy can reach it.
- The vite config has `server.open: true`, which spawns `xdg-open` and crashes in a
  headless container. The compose passes `--no-open` to disable it. Do NOT remove that
  flag, and do NOT re-enable `open` in the config.
- Vite 5.0.12 predates the Host-header allowlist feature, so no `allowedHosts` config is
  needed; `--host` alone binds 0.0.0.0.
- Deps are cached in anonymous volumes (`/opt/node_app/node_modules` and
  `excalidraw-app/node_modules`); first boot installs, later boots reuse them.

## Secrets
None required. All values in `.env.development` (Firebase config, library/AI/collab URLs)
are public client-side config already committed. The editor works offline; collaboration
and AI features point at localhost/remote services and are optional.

## Verify
`curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/` → `200`, and
`http://localhost:3000/src/index.tsx` → `200` (confirms live source, not a prebuilt bundle).
