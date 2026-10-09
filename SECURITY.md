# Security Policy

Thank you for taking the time to help keep **Uiua Resonance Lab** secure and trustworthy.

---

## Supported Versions

| Version | Supported |
|---|---|
| `0.14.x` (current) | ✅ Actively supported |
| `< 0.14` | ❌ Pre-release snapshots, no support |

Because this project is a static, client-side web application with no server-side
component, no persistent storage, no authentication, and no network-exposed
API surface, the attack surface is intentionally minimal. Security
considerations are dominated by client-side safety, supply-chain integrity of
CDN-delivered assets, and responsible disclosure practices.

---

## Scope

In-scope security concerns include:

- **Cross-site scripting (XSS)** via user-controlled input (glyph buffer, URL parameters, localStorage if added in the future).
- **Supply-chain risks** from CDN dependencies (Tailwind CSS, Google Fonts).
- **Sensitive data leakage** — although the app does not collect data, any future addition must honor this policy.
- **Denial of service** via pathological inputs (e.g., rendering resolution high enough to freeze the browser tab).
- **Mixed content** (HTTP assets loaded over HTTPS) on the deployment target.
- **Clickjacking** exposure on hosted deployments.
- **Sub-resource integrity (SRI)** gaps for CDN assets.

Out of scope:

- Vulnerabilities in third-party services this project links to but does not control (uiua.org, fonts.googleapis.com, cdn.tailwindcss.com — report to those vendors).
- Self-XSS where the attacker must convince a victim to paste malicious glyphs into the code buffer themselves (standard browser security model applies).
- Issues requiring physical access to the user's device.

---

## Reporting a Vulnerability

**Please do not file public GitHub issues for security vulnerabilities.**

Instead, report privately via:

- **GitHub Security Advisory**: use the *Report a Vulnerability* button on the [Security tab](https://github.com/zazieproductions/UIUA-RESONANCE-LAB/security) of this repository.
- **Email**: send details to the maintainer via the contact address on the GitHub profile.

### What to include

A useful report contains:

1. **Description** — summary of the issue and its impact.
2. **Steps to reproduce** — minimal, step-by-step instructions.
3. **Proof of concept** — code, HTML snippet, or HTML file demonstrating the issue.
4. **Impact assessment** — what an attacker could achieve.
5. **Suggested fix** (optional) — if you have one.
6. **Your contact** — for follow-up questions and coordinated disclosure.

### Response timeline

- **Acknowledgement:** within 72 hours.
- **Initial assessment & triage:** within 7 days.
- **Fix & disclosure:** coordinated with the reporter; we aim for public disclosure within 30 days of acknowledgement for confirmed issues, with credit to the reporter unless they request anonymity.

---

## Security Design Principles

The project is engineered with security conservatism as a deliberate property:

1. **No server.** The entire application runs in the user's browser; there is no backend to breach.
2. **No eval.** The glyph buffer is a *simulated* tacit interface — it does not use `eval()`, `new Function()`, or `innerHTML` insertion of user text. The only `innerHTML` usage is for static, developer-controlled strings with no user interpolation.
3. **No network requests at runtime** after the initial page load (aside from Google Fonts and Tailwind on first paint). No analytics, no tracking, no telemetry.
4. **No storage.** The application does not write to `localStorage`, `IndexedDB`, or cookies, preserving a stateless footprint.
5. **Explicit CORS-safe CDN usage.** External resources are loaded from reputable providers with `crossorigin` attributes where applicable.
6. **Autoplay policy compliance.** Audio is gated behind a user gesture, preventing unwanted audio exfiltration channels.

### Known hardening opportunities (tracked as issues)

- Add [Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) attributes to CDN `<link>` and `<script>` tags.
- Add a `Content-Security-Policy` meta tag that restricts script sources to `'self'` and approved CDNs.
- Add `X-Frame-Options: DENY` or equivalent `frame-ancestors` directive for clickjacking mitigation (host-dependent).

---

## Bug Bounty

This project does not currently offer a paid bug-bounty program. Recognition will be given in the [CHANGELOG](CHANGELOG.md) and release notes, with credit to the reporter (unless anonymity is requested).

---

Thank you for helping keep the creative-coding community safe.
