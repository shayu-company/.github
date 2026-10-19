# Shayou shared GitHub Actions

Organization-wide workflows and workflow templates.

## Reusable workflow

`.github/workflows/deploy-frontend.yml` builds a static frontend with Node.js 24 and deploys versioned releases over SSH/rsync. It accepts `environment` (staging or production), `node-version` and `dist-path`.

## GitHub environments (in the calling application repository)

Configure **staging** and **production** environments. Each requires:

| Type | Name | Meaning |
| --- | --- | --- |
| Variable | `DEPLOY_HOST` | VPS address |
| Variable | `DEPLOY_USER` | Restricted deploy account |
| Variable | `DEPLOY_PORT` | Optional SSH port, defaults to 22 |
| Variable | `DEPLOY_PATH` | Pre-provisioned directory under /var/www |
| Secret | `SSH_PRIVATE_KEY` | Dedicated SSH deploy key |
| Secret | `SSH_KNOWN_HOSTS` | Verified SSH host key entry |

Staging runs from the `staging` branch and production from `main`. A manual dispatch on either branch also works.

This initial workflow **does not provision Nginx, DNS, SSL certificates or server permissions**. The server must already expose the application directory and Nginx must serve the `current` symlink. The workflow uploads to `releases/<sha>-<run_id>` then switches `current` after upload. Retention/rollback management is a later step.

## Using the template

The `workflow-templates/` directory provides a GitHub Actions starter. Alternatively, create a repository from the private `shayu-company/template-frontend` template, which already includes its caller workflow and CI.

For production use, pin shared workflows and third-party actions to immutable revisions. Never commit SSH keys. Verify host fingerprints independently rather than blindly trusting ssh-keyscan.

## Full-stack Node.js starter

The organization also offers `workflow-templates/fullstack-deploy.yml`, with selective frontend/backend validation and deployment for `staging` and `main`. It is offered alongside the existing static-frontend template and does not modify it.

**Important:** Before using this starter, copy `workflow-templates/fullstack-ci.yml.example` to `.github/workflows/ci.yml` in the application repository. The deployment starter calls it using `uses: ./.github/workflows/ci.yml`. The starter assumes `frontend/` (with `check`, `format:check`, `test`, `build`, `smoke:build` scripts), `backend/` (`npm test`, `src/server.js`), Node.js 24, and a `frontend/dist` build output. Adjust the sample for different frameworks or layouts.

For each GitHub Environment (`staging` and `production`), configure variables `DEPLOY_HOST`, `DEPLOY_USER`, optional `DEPLOY_PORT`, `FRONTEND_PATH`, `BACKEND_PATH`, `API_PORT`, and `PM2_APP_NAME`; configure secrets `SSH_PRIVATE_KEY` and `SSH_KNOWN_HOSTS`. Deployment paths must exist under `/var/www`, and the server must already have Nginx, PM2, Node.js and any `.env` configured. Backend deployment preserves the existing `.env` and `node_modules` paths. This starter is a starting point, not a drop-in production deployment: validate the generated workflow and smoke-test before deploying, especially for database migrations and API compatibility.
