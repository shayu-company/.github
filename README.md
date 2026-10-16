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
