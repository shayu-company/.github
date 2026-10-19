# Shayu Company — GitHub Actions

Reusable deployment workflows and GitHub Actions workflow starters for Shayu projects.

## Supported application types

| Application template | GitHub Pages | VPS / other SSH server |
| --- | --- | --- |
| `template-static` (HTML / CSS / JS) | Yes | Yes |
| `template-angular` | Static build only | Yes |
| `template-static-node` (static + Express) | No | Yes |
| `template-angular-node` (Angular + Express) | No | Yes |

**Hosting is exclusive per application:** a GitHub Pages application is frontend-only. A server-hosted application may contain frontend, backend, or both. Do not generate a GitHub Pages frontend + VPS backend combination.

## Repository naming convention

- Application starters: `template-static`, `template-angular`, `template-static-node`, `template-angular-node`.
- Central workflows: `.github/workflows/deploy-pages.yml`, `deploy-frontend.yml`, `deploy-node.yml`.
- GitHub Actions starter templates: `workflow-templates/pages.yml`, `vps-static.yml`, `vps-fullstack.yml`, each with a matching `.properties.json`.

## Standard workflow layout in generated repositories

```text
.github/workflows/
  ci.yml
  deploy-staging.yml
  deploy-production.yml
```

Jobs follow Control Room conventions: `detect-changes`, `ci-frontend`, `ci-backend`, `deploy-frontend`, `deploy-backend` (omit irrelevant jobs). The target application template owns the stack-specific build and validation steps. Deployment uses reusable workflows in this repository where appropriate.

- GitHub Pages uses `main` and `deploy-production.yml`. Staging is optional and not provisioned by default. Configure **Settings → Pages → Build and deployment → GitHub Actions**.
- VPS applications use `staging` and `main` for deployments to separate GitHub Environments. Never deploy production from staging.
- Starter workflows in `workflow-templates/` are examples; a full-stack starter requires its own `ci.yml` and production equivalent in the application template. They are not a replacement for those files.

## Reusable workflows

- `deploy-pages.yml`: consumes an artifact uploaded with `actions/upload-pages-artifact@v3` and deploys it with `actions/deploy-pages@v4`. Caller must grant `pages: write` and `id-token: write`.
- `deploy-frontend.yml`: Node 24 build then SSH/rsync release deployment to a `current` symlink. Caller GitHub Environment vars: `DEPLOY_HOST`, `DEPLOY_USER`, `DEPLOY_PATH`, optional `DEPLOY_PORT`; secrets `SSH_PRIVATE_KEY`, `SSH_KNOWN_HOSTS`. The destination must already be provisioned.
- `deploy-node.yml`: deploys an Express application via SSH/rsync and restarts PM2. Caller vars: `DEPLOY_HOST`, `DEPLOY_USER`, `BACKEND_PATH`, `API_PORT`, `PM2_APP_NAME`, optional `DEPLOY_PORT`; secrets `SSH_PRIVATE_KEY`, `SSH_KNOWN_HOSTS`. The example assumes `backend/src/server.js`, `npm ci` and Node installed on the server; adjust for TypeScript/dist projects.

Do not place credentials in repository templates. Verify SSH host fingerprints independently, protect production environments, and pin third-party actions and reusable workflows to immutable revisions when hardening for production.

## Existing applications

`vchaillo/control-room` and `vchaillo/velomassifmorvan` use local GitHub Actions workflows, with no calls to `shayu-company/.github` in the workflows reviewed on 19 October 2026. They have not been migrated; their functioning deployment pipelines remain untouched.

The old `workflow-templates/frontend-deploy.*` and `fullstack-deploy.*` starters have been replaced by destination-oriented names. The historical `.github/workflows/deploy-frontend.yml` reusable workflow is retained to preserve compatibility.
