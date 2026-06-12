# DevSecOps Audit Report — Repository Hygiene & Credential Surface

> **Date:** 2026-06-11  
> **Auditor:** DevSecOps automated scan  
> **Scope:** Full repository (`client/`, `server/`, root) — untracked files, tracked artifacts, credential leakage, build noise, OS clutter  

---

## 1. Executive Summary

| Category | Status | Notes |
|----------|--------|-------|
| Hardcoded secrets (API keys, passwords, tokens) | ✅ **Clean** | No secret material found in source files. |
| `.env` / environment files in Git | ⚠️ **Partial risk** | `client/.env.production` is tracked and contains a real production URL. |
| `node_modules` in Git | ✅ **Clean** | Present on disk but correctly ignored. |
| OS artifacts (`.DS_Store`, `Thumbs.db`) | ✅ **Clean** | None found. |
| Build artifacts (`dist/`, `build/`) | ✅ **Clean** | None found outside `node_modules`. |
| Generated audit reports | ⚠️ **Untracked** | `audit.md` and `audit_fix.md` are not committed but float in the working tree. |
| Kimchi internal state | ⚠️ **Untracked** | `.kimchi/ferments` is present and untracked. |
| Dockerfile / `.dockerignore` | 🔴 **Missing** | Neither file exists. Container builds will ship unnecessary files. |

---

## 2. Detailed Findings

### 2.1 Git Working Tree State

```
 M client/src/App.jsx
 M server/package-lock.json
 M server/server.js
?? audit.md
?? audit_fix.md
```

**Modified files**
- `client/src/App.jsx` — pending frontend change.
- `server/package-lock.json` — dependency drift (expected after installs).
- `server/server.js` — pending backend change.

**Untracked files**
- `audit.md` — generated deployment audit.
- `audit_fix.md` — generated fix log.
- `.kimchi/ferments/` — internal Kimchi orchestration state.

---

### 2.2 Environment & Configuration Files

| File | Tracked? | Risk Level | Reason |
|------|----------|------------|--------|
| `client/.env.example` | ✅ Yes | 🟢 Low | Safe template; no real values. |
| `server/.env.example` | ✅ Yes | 🟢 Low | Safe template; no real values. |
| `client/.env.production` | ✅ **Yes** | 🟡 **Medium** | Contains real production backend URL (`https://dev-resource-tracker-api.onrender.com/api`). Not a cryptographic secret, but leaks deployment topology and makes environment promotion harder. |

**Why `client/.env.production` is dangerous**
- `.gitignore` currently blocks `.env`, `*.env`, and `.env.*.local`, but **not** `.env.production`.
- In Vite, `.env.production` is auto-loaded during `vite build` and its `VITE_` variables are baked into the static bundle.
- Committing this file to Git means every clone knows your production API endpoint. It also forces all builds of this commit to point to the same backend, regardless of the actual target environment (staging, QA, demo).

---

### 2.3 Hardcoded Infrastructure References

| File | Line | Hardcoded Value | Risk |
|------|------|-----------------|------|
| `server/server.js` | 17 | `https://dev-resource-tracker-api.netlify.app` | CORS whitelist. Domain name contains `api`, suggesting this may be the backend URL copy-pasted into the frontend slot. If the real frontend lives elsewhere, CORS will block it. |
| `server/server.js` | 17 | `http://localhost:5173` | CORS whitelist. Only valid when both frontend and backend run on the same host. Breaks Docker networking, remote dev containers, and CI preview deployments. |
| `server/server.js` | 98 | `` `📍 Local: http://localhost:${PORT}` `` | Log line only; harmless but leaks the assumption that the operator is on localhost. |

**Verdict:** No cryptographic secrets, but the combination of tracked `.env.production` + hardcoded origin whitelist leaks enough information to fingerprint the deployment architecture.

---

### 2.4 Build Noise & Dependency Artifacts

| Artifact | Location | Ignored? | Size (approx) |
|----------|----------|----------|---------------|
| `node_modules/` | `server/node_modules/` | ✅ Yes by `.gitignore` | ~30–80 MB |
| Nested `node_modules/` | Inside `server/node_modules/*/node_modules/` | ✅ Yes (inherited) | — |
| `package-lock.json` | `client/package-lock.json` | ⚠️ Tracked | ~100 KB |
| `package-lock.json` | `server/package-lock.json` | ⚠️ Tracked (modified) | ~80 KB |

**Note on lockfiles:**  
`package-lock.json` is intentionally tracked in this repo. Modern Node.js best practice is to commit lockfiles for reproducible builds. This is **not a security issue**, but the modified server lockfile indicates uncommitted dependency drift that should be resolved before deployment.

---

### 2.5 Missing Docker Guardrails

| Missing File | Impact |
|--------------|--------|
| `.dockerignore` | A `docker build` from this repo root will copy `node_modules`, `.git`, `.env`, audit reports, IDE folders, and OS files into the build context. This bloats the image, slows builds, and increases the chance that a secret file accidentally lands in a layer. |
| `Dockerfile` | No container definition exists for either the client or the server. |

---

### 2.6 Kimchi Tooling State (Untracked)

- `.kimchi/ferments/` exists in the working tree but is untracked.
- This directory may contain host-side orchestration state, temporary plans, or session metadata.
- **Risk:** If a contributor accidentally runs `git add -A` or `git add .`, this folder could be committed, leaking local session context.

---

## 3. Recommended Additions

### 3.1 Append to Root `.gitignore`

Copy the block below into the end of your root `.gitignore` file:

```gitignore
# ---------------------------------------------------------------------------
# DevSecOps additions — 2026-06-11
# ---------------------------------------------------------------------------

# Production environment files (contain real deployment URLs / endpoints).
# These must NOT be committed because Vite bakes VITE_* variables into the
# static bundle at build time, freezing the target environment.
.env.production
*/.env.production
.env.staging
*/.env.staging

# Generated audit / analysis reports (ephemeral artifacts).
audit*.md
!README.md

# Kimchi internal orchestration state (host-specific, non-portable).
.kimchi/

# Local Docker overrides that may contain secrets or host paths.
docker-compose.override.yml
*.override.yml
```

**What this blocks:**
- Any `.env.production` at root or in subprojects.
- Any markdown file beginning with `audit` (e.g. `audit.md`, `audit_fix.md`).
- The `.kimchi/` directory recursively.
- Common Docker override files that often hold local credentials.

---

### 3.2 Create `.dockerignore` (does not exist)

Create a file named `.dockerignore` in the **repository root** with the following exact content:

```dockerignore
# ---------------------------------------------------------------------------
# .dockerignore — Dev Resource Tracker
# Prevents build-context bloat and accidental secret leakage in layers.
# ---------------------------------------------------------------------------

# Git history
.git
.gitignore

# Node dependency trees (re-installed inside the image)
node_modules
*/node_modules
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*

# Lockfiles — policy decision:
# If you install inside Docker from package.json only, uncomment the next two lines.
# If you copy lockfiles for reproducible builds, leave them commented.
# package-lock.json
# */package-lock.json

# Environment & secrets
.env
.env.*
!.env.example

# Build artifacts (re-generated inside image or served externally)
dist
*/dist
build
*/build
dist-ssr

# Coverage & test output
coverage
*/coverage
*.lcov

# OS & IDE clutter
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db
.vscode
.idea
*.swp
*.swo
*.suo
*.ntvs*
*.njsproj
*.sln

# Audit / generated docs
audit*.md
README.md

# Kimchi tooling state
.kimchi

# Temporary folders
tmp
temp
*.tmp
*.temp
```

**How to use it:**
1. Save the file as `.dockerignore` in the repo root.
2. Ensure it sits next to your future `Dockerfile`.
3. Run `docker build` from the repo root; the `.dockerignore` is picked up automatically.

---

## 4. Immediate Action Checklist

- [ ] **Review** `client/.env.production` — decide whether to `git rm --cached` it and move the value into CI/CD secrets or a deployment manifest.
- [ ] **Append** the `.gitignore` block above to the root `.gitignore`.
- [ ] **Create** `.dockerignore` with the template above.
- [ ] **Verify** no accidental `.env` files are tracked:
  ```bash
  git ls-files | grep -E '\.env(\.[^/]+)?$'
  ```
  Expected result: only `.env.example` files should appear.
- [ ] **Clean** the working tree:
  ```bash
  # Remove generated reports from tracking consideration
  echo "audit*.md" >> .gitignore
  echo ".kimchi/" >> .gitignore
  ```
- [ ] **Stage & commit** the `.gitignore` and `.dockerignore` changes:
  ```bash
  git add .gitignore .dockerignore
  git commit -m "chore: add DevSecOps ignore rules for env files and build noise"
  ```

---

*End of DevSecOps Audit*
