# Security Policy

## Reporting a Vulnerability

To report a security vulnerability, please **do not** file a public GitHub issue.

Use [GitHub's private vulnerability reporting](https://github.com/patriciobcs/openreader-support/security/advisories/new)
or email **security@openreader.io** with:

- A description of the vulnerability
- Steps to reproduce
- Potential impact

We aim to respond within 72 hours and will coordinate a fix and responsible disclosure timeline with you.

## Coordinated Disclosure

We follow a 90-day coordinated disclosure policy. After 90 days from your report (or when a fix is shipped, whichever is sooner), we ask that you publish your findings. We will credit you in the release notes unless you prefer otherwise.

## Scope

The following are in scope for security reports:

- OpenReader desktop app (Tauri + Python TTS backend at `apps/desktop/`)
- OpenReader web app (Next.js at `apps/web/`)
- Shared reader package (`packages/reader/`)
- Python TTS backend endpoints (`/tts/generate`, `/tts/stream`, `/models/*`)

The following are out of scope:

- Third-party TTS model weights downloaded from HuggingFace
- Issues already tracked in upstream projects (Tauri, Next.js, pdfjs-dist, MLX)
- Vulnerabilities requiring physical access to the device

## Automated Dependency Monitoring

Dependency security is monitored automatically via GitHub Dependabot (see `.github/dependabot.yml`). Dependabot scans:

- JavaScript/TypeScript (npm/Bun) - root, `apps/web`, `apps/desktop`
- Rust (Cargo) - `apps/desktop/src-tauri`
- Python (pip/uv) - `apps/desktop`
- GitHub Actions - root `.github/workflows/`

Security update PRs are opened individually (not grouped) for immediate visibility. Version-update PRs are grouped weekly to reduce noise.

## CI Security Baseline

Pull requests and pushes to `main` run the non-GUI gate in
`.github/workflows/non-gui-gate.yml`. Every run executes the fast gate:

```bash
bun run check:fast
```

Pushes to `main`, manual dispatch, and pull requests labelled
`full-acceptance` also run the `check:ci` contract in parallel jobs: repository
guard tests and TypeScript coverage, backend Ruff lint and pytest coverage
without public-network tests, and the combined coverage policy. GUI, real MLX, notarization, and
release signing gates remain opt-in release evidence because they require
platform hardware, cached model weights, or private signing credentials.

## Known Unaddressed Advisories

The 2026-10-20 launch preflight updated Next.js to 16.3.8, DOMPurify to
3.4.17, sharp to 0.35.5, and the affected locked transitive dependencies.
`bun audit` then reported no vulnerabilities across 307 packages. This is a
dated JavaScript dependency result, not an audit of every native or model
dependency. The Next.js update includes the upstream
[image-optimization SSRF fix](https://github.com/advisories/GHSA-cjq9-62q9-8jv4)
and [ImageResponse code-execution fix](https://github.com/advisories/GHSA-vcvr-r3jv-pc5j).

The full Git-history Gitleaks scan found no credentials after reviewing one
false positive: the literal audio-format identity comparison in historical
mobile code. `.gitleaksignore` records only that exact commit/file/rule/line
fingerprint; other findings remain enabled.

The following advisories are tracked but not immediately actionable. Each will be resolved when the upstream fix reaches a stable release compatible with our dependency tree.

### Rust (Cargo) - Tauri Linux backend warnings

These are `unmaintained` and `unsound` warnings for the GTK3 bindings pulled in transitively by Tauri/wry on Linux. They do NOT apply to the macOS app (which uses WebKit directly, bypassing GTK entirely).

| Advisory | Package | Type | Blocker |
|---|---|---|---|
| RUSTSEC-2024-0411 through RUSTSEC-2024-0420 | `atk`, `gdk`, `gtk`, `gdkwayland-sys`, `gdkx11`, `gtk-sys`, `gtk3-macros`, `gdk-sys`, `atk-sys`, `gdkx11-sys` | unmaintained (gtk-rs GTK3) | Blocked on Tauri upgrading its Linux renderer from gtk3 to gtk4/libadwaita. No ETA; tracked upstream at tauri-apps/tauri. |
| RUSTSEC-2024-0370 | `proc-macro-error` | unmaintained | Pulled in by `gtk3-macros`; same blocker as above. |
| RUSTSEC-2024-0429 | `glib` | unsound (VariantStrIter Iterator impl) | Same gtk-rs chain as above; not exploitable without unsafe Rust in user code. |
| RUSTSEC-2025-0075, RUSTSEC-2025-0080, RUSTSEC-2025-0081, RUSTSEC-2025-0098, RUSTSEC-2025-0100 | `unic-char-range`, `unic-common`, `unic-char-property`, `unic-ucd-version`, `unic-ucd-ident` | unmaintained | Pulled in by `urlpattern` via `tauri-utils`. Upstream Tauri is expected to remove the unic dependency in a future release. |

### JavaScript - None

All known JS/TS vulnerabilities have been resolved as of the baseline audit (2026-05-01). See the bump list below.

### Python - None

No known Python vulnerabilities as of the baseline audit (2026-05-01).

## Dependency Versions Fixed in Baseline Audit (2026-05-01)

The following packages were upgraded to resolve high/moderate CVEs:

| Package | Old Version | New Version | CVEs Resolved |
|---|---|---|---|
| `vite` | 8.0.3 | 8.0.10 | GHSA-4w7w-66w2-5vf9 (moderate, .map path traversal), GHSA-v2wj-q39q-566r (high, server.fs.deny bypass), GHSA-p9ff-h696-f583 (high, dev server arbitrary file read via WebSocket) |
| `next` | 16.2.1 | 16.2.4 | GHSA-q4gf-8mx6-v5v3 (high, Server Components DoS) |
| `yaml` | 2.8.2 | 2.8.4 | GHSA-48c2-rrv3-qjmp (moderate, Stack Overflow via deeply nested YAML) |
| `picomatch` | 4.0.3 | 4.0.4 | GHSA-3v7f-55p6-f55p (moderate, Method Injection in POSIX Character Classes), GHSA-c2c7-rcm5-vvqj (high, ReDoS via extglob quantifiers) |
| `postcss` | 8.4.31 / 8.5.8 | 8.5.13 | GHSA-qx2v-qp2m-jg93 (moderate, XSS via unescaped `</style>` in CSS Stringify) |
