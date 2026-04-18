# `get.goldenfocus.io` — install.sh gateway

A 40-line Cloudflare Worker that serves `curl -fsSL get.goldenfocus.io | bash` from the startup kit in this repo.

## What it does

| Path | Serves |
|---|---|
| `/` | `startup-kit/install.sh` |
| `/elite`, `/kit` | same (aliases) |
| `/raw/<anything>` | raw file from this repo at `<anything>` |
| anything else | tries to resolve in repo root |

Caches upstream for 5 minutes; edits to `install.sh` propagate within that window.

## Deploy

```bash
cd get/
npx wrangler login           # browser OAuth, once per machine
npx wrangler deploy          # creates/updates the Worker + route
```

After deploy, `curl https://get.goldenfocus.io` returns the install script.

## Requirements

- `goldenfocus.io` zone exists on the authenticated Cloudflare account (it already does — Cloudflare NS are in place)
- If there's a pre-existing DNS record for `get.goldenfocus.io`, Cloudflare will honor the Worker route over it. If the record blocks deployment, delete the stale record first:
  ```bash
  # list records; find the stale "get" one; delete it
  npx wrangler dns list --zone goldenfocus.io
  ```

## Rollback

```bash
npx wrangler delete golden-focus-get
```

Route removes itself; DNS falls back to whatever (if anything) existed before.
