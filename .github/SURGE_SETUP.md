# GitHub Actions Deployment Setup

This repository is configured to automatically deploy to Surge.

## Security Model

Our CI/CD pipeline uses a **two-stage workflow_run pattern** that safely handles untrusted PR code:

### How It Works

```
Stage 1: Build Workflow (build.yml)
- Triggers on: push to main, pull_request
- Permissions: contents: read (NO secrets access for fork PRs)
- Output: Static artifacts (dist/)

                          |
                          v workflow_run trigger

Stage 2: Deploy Workflow (deploy-surge.yml)
- Triggers on: workflow_run completed
- Permissions: HAS secrets access
- Runs: Download artifact -> Deploy static files
- Key: Never executes PR code, only deploys pre-built artifacts
```

## Setup Instructions

### 1. Get Your Surge Token

First, login to Surge locally (one-time setup):

```bash
npm install -g surge
surge login
```

Then get your token:

```bash
surge token
```

### 2. Create GitHub Environment

1. Go to your GitHub repository
2. Navigate to **Settings** -> **Environments**
3. Click **New environment**
4. Name it `surge-deploy`

### 3. Add GitHub Secrets

In the `surge-deploy` environment, add these secrets:

#### Secret 1: SURGE_TOKEN

- **Name**: `SURGE_TOKEN`
- **Value**: Your token from `surge token` command

#### Secret 2: SURGE_DOMAIN

- **Name**: `SURGE_DOMAIN`
- **Value**: `your-app-name.surge.sh`

### 4. Push to GitHub

Once secrets are configured, any push to `main` will trigger:

1. Build workflow uploads artifacts
2. Deploy workflow deploys to Surge

### 5. Verify Deployment

After pushing, check:

- **GitHub Actions**: See the workflow run in the Actions tab
- **Live Site**: Visit your surge domain

## Workflow Details

The workflow runs on:

- Every push to `main` branch -> deploys to production
- Pull requests to `main` -> deploys to `pr-N-your-domain.surge.sh`

### Manual Deployment

You can still deploy manually:

```bash
just deploy-prod
```

## Troubleshooting

### Deployment Fails

- Verify `SURGE_TOKEN` is valid: `surge token`
- Check `SURGE_DOMAIN` format (no `https://` prefix)
- Ensure secrets are properly set in GitHub environment
