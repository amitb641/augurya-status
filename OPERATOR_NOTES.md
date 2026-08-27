# Operator notes — augurya-status

## DNS (required before the page is publicly reachable)

The site is built and deploying correctly to the `gh-pages` branch (GitHub Pages
is enabled, sourced from `gh-pages`, and correctly picked up the custom domain
from the site's own CNAME file). It is **not yet reachable** — `status.augurya.com`
needs a DNS CNAME record pointing at `amitb641.github.io`, added wherever
Augurya's other DNS is managed. This is the one remaining manual step; nothing
else is blocked on it.

Once the CNAME resolves, also flip HTTPS enforcement on (GitHub can't issue a
cert for the custom domain until DNS actually resolves to Pages):

```
gh api -X PUT repos/amitb641/augurya-status/pages -f https_enforced=true
```

## Weekly template auto-sync (`update-template.yml` / the "Update template"
## step inside `setup.yml`) will fail until a `GH_PAT` secret is added

Confirmed live (2026-08-27): the default `GITHUB_TOKEN`, even with
`default_workflow_permissions: write` set at the repo level, can **never**
push changes to `.github/workflows/*` — this is a hard GitHub platform
restriction (fine-grained PAT or classic PAT with the `workflow` scope is the
only way around it), not something a repo setting can override. Both
`setup.yml` and `update-template.yml` are written to expect this: they already
fall back to `secrets.GH_PAT || github.token`, and Upptime's own docs document
adding a `GH_PAT` secret as the standard setup step for exactly this reason.

The actual monitoring (`uptime.yml`, every 5 min) and the site build/deploy
(`site.yml`, daily + on `assets/**` changes) do **not** touch
`.github/workflows/*` and work fine with the default token — verified live,
both ran clean on 2026-08-27. Only the weekly workflow-file sync is affected;
until `GH_PAT` is added, that one step will fail weekly (harmlessly -- it
only means new Upptime template releases won't auto-apply here) rather than
being silently skipped.

To fix: create a fine-grained PAT scoped to only this repo with `Contents:
Read and write` + `Workflows: Read and write`, then add it as a repository
secret named `GH_PAT` (Settings -> Secrets and variables -> Actions -> New
repository secret). This is an account-level action -- not something an
agent session can do on your behalf.

## What's monitored

Three public, customer-facing surfaces only (deliberately excludes internal
tools -- staff shell, Quant Lab):
- `https://augurya.com` (marketing site)
- `https://app.augurya.com` (member shell)
- `https://api.augurya.com/api/health` (backend health)

Notifications are not configured. Add a channel (email/Slack/etc.) via
`.upptimerc.yml`'s `status-website` notification config when you want alerts
on top of the public status page -- see https://upptime.js.org/docs/configuration.
