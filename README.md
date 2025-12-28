# Hello Surge

A minimal hello world app with Surge deployment via GitHub Actions.

**Live Site**: https://hello-surge.surge.sh (once configured)

## Setup Surge Deployment

### 1. Install Surge CLI

```bash
npm install -g surge
surge login
```

### 2. Get Your Surge Token

```bash
surge token
```

Copy the output token.

### 3. Create GitHub Environment

1. Go to repo **Settings** → **Environments**
2. Click **New environment**
3. Name it `surge-deploy`

### 4. Add Secrets to the Environment

In the `surge-deploy` environment, add:

| Secret | Value |
|--------|-------|
| `SURGE_TOKEN` | Your token from `surge token` |
| `SURGE_DOMAIN` | `hello-surge.surge.sh` |

### 5. Deploy

Push to `main` and the workflow will automatically deploy.

```bash
git push origin main
```

### Manual Deploy

```bash
just deploy-prod
```

## How It Works

The deployment uses a secure two-stage workflow:

1. **Build workflow** (`build.yml`): Runs on PRs/pushes, uploads `dist/` as artifact
2. **Deploy workflow** (`deploy-surge.yml`): Triggered by build completion, deploys artifact to Surge

This pattern is secure because:
- Fork PRs run build WITHOUT secrets
- Deploy only downloads pre-built artifacts (no code execution)
- PR previews auto-deploy to `pr-N-hello-surge.surge.sh`
- PR previews auto-teardown when PR closes

## Adding PWA Support

To make your app installable and work offline:

### 1. Install Dependencies

```bash
npm install vite vite-plugin-pwa
```

### 2. Create `vite.config.ts`

```typescript
import { defineConfig } from 'vite';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    VitePWA({
      registerType: 'autoUpdate',
      includeAssets: ['favicon.ico', 'icons/*.png'],
      manifest: {
        name: 'Your App Name',
        short_name: 'AppName',    // max 12 chars
        description: 'Your app description',
        theme_color: '#667eea',
        background_color: '#667eea',
        display: 'standalone',
        orientation: 'portrait',
        icons: [
          {
            src: 'icons/icon-192.png',
            sizes: '192x192',
            type: 'image/png',
          },
          {
            src: 'icons/icon-512.png',
            sizes: '512x512',
            type: 'image/png',
            purpose: 'any maskable',
          },
        ],
      },
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}'],
      },
    }),
  ],
  build: {
    outDir: 'dist',
  },
});
```

### 3. Add Icons

Create `public/icons/` with:
- `icon-192.png` (192x192)
- `icon-512.png` (512x512)

### 4. Update Build

```bash
npm run build   # Now outputs PWA-enabled dist/
```

### PWA Success Criteria

Your PWA should:
- Show browser "Install" prompt
- Work offline after first visit
- Detect and notify about updates
- Run fullscreen when installed

See [PWA Enablement Spec](https://github.com/idvorkin/chop-conventions/blob/main/pwa/PWA_ENABLEMENT_SPEC.md) for full implementation details.

## File Structure

```
├── dist/
│   └── index.html          # Your static site
├── .github/
│   ├── workflows/
│   │   ├── build.yml       # Build and upload artifact
│   │   └── deploy-surge.yml # Deploy to Surge
│   └── SURGE_SETUP.md      # Detailed setup docs
├── justfile                 # Local commands
└── README.md
```
