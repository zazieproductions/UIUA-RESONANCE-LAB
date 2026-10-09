# Deployment

Uiua Resonance Lab is a static web application: a single `index.html` file plus zero build artifacts. This means deployment reduces to "serve this folder over HTTP(S)." There are many ways to do this; this document covers the most common targets, cache strategy, and continuous-deployment considerations.

---

## Deployment Targets (Compatibility)

The application has no server-side component. Any static file host works. In approximate order of popularity:

| Host | Build Config | HTTPS | Custom Domain | Notes |
|---|---|---|---|---|
| **GitHub Pages** | None (push to branch) | ✅ | ✅ | Recommended default; zero-cost for public repos |
| **Netlify** | Drag-and-drop or Git | ✅ | ✅ | Instant previews per PR |
| **Vercel** | Zero-config | ✅ | ✅ | Fast edge network |
| **Cloudflare Pages** | Zero-config | ✅ | ✅ | Free tier generous; integrates with R2 |
| **Surge.sh** | CLI one-liner | ✅ | ✅ | `surge .` from the repo root |
| **S3 + CloudFront** | Manual | ✅ via CF | ✅ | Enterprise-scale; requires AWS config |
| **Firebase Hosting** | `firebase deploy` | ✅ | ✅ | Google ecosystem |
| **Any HTTP server** | Place folder in docroot | varies | varies | `python3 -m http.server` for local, nginx/apache for production |

---

## GitHub Pages Deployment (Recommended)

Because the project is hosted on GitHub, GitHub Pages is the simplest production deployment.

### Option A — Deploy from `main` branch
1. Repository Settings → Pages.
2. Source: **Deploy from a branch**.
3. Branch: `main`, folder: `/ (root)`.
4. Save. The site will be available at `https://zazieproductions.github.io/UIUA-RESONANCE-LAB/` within a minute.
5. (Optional) Set a custom domain by adding a `CNAME` file and configuring DNS per GitHub docs.

### Option B — GitHub Actions Workflow (More Control)
For future-proofing (if a build step is ever added, or to support minification/SRI), a `.github/workflows/deploy.yml` can automate the deploy:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - id: deployment
        uses: actions/deploy-pages@v4
```

This workflow simply uploads the repository root as the Pages artifact and deploys it. No build step is performed, preserving the zero-build guarantee.

> **Note:** GitHub Pages serves `index.html` at the root automatically. Because the entry file was renamed from `UIUA RESONANCE LAB.html` to `index.html` for canonical web serving, this works without configuration.

---

## Cache Strategy

The application has no fingerprinted asset filenames (because there is no build), so cache policy must be conservatively short:

| File | Cache-Control | Rationale |
|---|---|---|
| `index.html` | `Cache-Control: no-cache` (or `max-age=0, must-revalidate`) | Users should always get the latest version on reload |
| `docs/` (Markdown) | `Cache-Control: public, max-age=300` (5 min) | Documentation changes less frequently than the app |
| `*.md`, `LICENSE`, etc. | `Cache-Control: public, max-age=86400` (24 h) | These rarely change |

Most static hosts (GitHub Pages, Netlify, Vercel) set sensible defaults. For custom nginx/apache/S3 configurations, tune cache headers explicitly.

CDN assets (Tailwind, Google Fonts) are served from their respective CDNs with long-lived caches and ETags — no configuration needed on your end.

---

## HTTPS & Security Headers

GitHub Pages, Netlify, Vercel, and Cloudflare Pages all enable HTTPS by default. If deploying behind your own server, ensure:

- **HTTPS only** — redirect HTTP → HTTPS via 301.
- **HSTS** header: `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`.

Recommended security headers for the response serving `index.html`:

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.tailwindcss.com 'unsafe-inline'; style-src 'self' https://cdn.tailwindcss.com https://fonts.googleapis.com 'unsafe-inline'; font-src https://fonts.gstatic.com; img-src 'self' data:; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'none'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=(), interest-cohort=()
```

> **Note:** Tailwind CDN applies styles via inline `<style>` injection at runtime, which is why `'unsafe-inline'` for `style-src` is currently required. Migrating to a built CSS file (or self-hosted Tailwind CSS) would allow removing `'unsafe-inline'`. Tracked as a future hardening item.

---

## Local / Offline Use

The application is fully functional when served from `file://` (double-clicking `index.html`), with the following caveats:
- Some browsers restrict Web Audio from `file://` contexts differently; if audio does not initialize, use a local HTTP server (see Quick Start in README).
- CDN resources (Tailwind, Google Fonts) require internet access on first load. After the browser caches them, subsequent loads work offline. To make the app fully offline-capable, self-host the Tailwind CSS and font files; this is tracked on the roadmap.

There is no service worker in v0.14 — adding one would enable offline-first behavior but introduces update-complexity that's premature for the current version.

---

## Path Resolution

Because the app is a single HTML file with no relative asset imports, **it works correctly when hosted at any subpath**:
- At root (`https://example.com/`)
- At a subpath (`https://example.com/uiua/`)
- At GitHub Pages project URLs (`https://zazieproductions.github.io/UIUA-RESONANCE-LAB/`)

There are no `fetch()` calls or hardcoded absolute paths that would break under subpath hosting.

---

## Performance Considerations for Production

- **Compression.** All major static hosts automatically apply gzip/brotli compression for HTML/JS/CSS. The 37 KB `index.html` compresses to ~10 KB with gzip.
- **HTTP/2 or HTTP/3.** All major static hosts support this by default; multiplexing ensures CDN assets load in parallel.
- **Preconnect.** The HTML already includes `<link rel="preconnect">` for Google Fonts, saving one RTT on cold load.
- **No server-side rendering.** The page is a static file; TTFB is the time to serve a single cached HTML file (~50 ms globally on edge hosts).

---

## Monitoring & Analytics

v0.14 ships with **no analytics or tracking** — no Google Analytics, no Plausible, no Sentry, no telemetry. This is a deliberate privacy-respecting choice. If future versions introduce error monitoring (e.g., Sentry for JS errors), it should:
- Be gated behind an opt-in (no silent pings).
- Be documented in SECURITY.md and README.
- Not collect personally identifiable information.

Referrer-Policy and Permissions-Policy headers are set to minimize accidental data leakage.

---

## Production Release Checklist

Before cutting a release (tagging a version and deploying):

1. All changes merged into `main`.
2. Manual test checklist passes on at least two P0 browsers (see [TESTING.md](TESTING.md)).
3. CHANGELOG.md updated with the version section.
4. Version badge in README and header badge (`v0.14`) updated to match.
5. Git tag created: `git tag -a v0.14.0 -m "v0.14 — Tacit Array"` and pushed.
6. GitHub Release created from the tag, with CHANGELOG entries as release notes.
7. Verify the deployed site loads and audio functions at the production URL.

---

## Custom Domain Notes

When pointing a custom domain at GitHub Pages (or other static host):

1. Set the CNAME / A records per host documentation.
2. Ensure HTTPS is provisioned (GitHub Pages provisions via Let's Encrypt automatically within minutes).
3. Add the CNAME file to the repo root for GitHub Pages custom domain persistence:
   ```
   your-domain.com
   ```
4. Enable "Enforce HTTPS" in repository Pages settings.
