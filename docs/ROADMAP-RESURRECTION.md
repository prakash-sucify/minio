# MinIO Fork Roadmap: Supply Chain Continuity & Console Restoration

This roadmap aligns with the community resurrection approach described in [MinIO Is Dead, Long Live MinIO](https://blog.vonng.com/en/db/minio-resurrect/?ref=dailydev). The goal is **supply chain continuity**: deliver a working, CVE-patched MinIO build with the full admin console, without adding new features.

---

## Current State (as of March 2026)

| Area | Status |
|------|--------|
| **Fork** | `prakash-sucify/minio` — `origin`; `minio/minio` — `upstream` |
| **Console** | Go dependency `github.com/minio/console` (current). Full CE restoration: see [docs/CONSOLE.md](CONSOLE.md). |
| **Binaries** | ✅ Release workflow builds linux/amd64, linux/arm64 and uploads to GitHub Releases with SHA256. |
| **Docker** | ✅ `Dockerfile.build` builds from source (no dl.min.io). CI publishes to `ghcr.io/<owner>/minio`. |
| **CVE / vuln** | `vulncheck.yml` runs `govulncheck` on PR/push — good baseline. |
| **Docs** | ✅ README fork overview, CONSOLE.md, CONTRIBUTING policy, trademark disclaimer. |

---

## Phase 1: Restore Full Admin Console

**Objective:** Restore the full MinIO admin console (user management, bucket policies, access control, lifecycle) that was removed from CE in May 2025.

### 1.1 Identify correct console version

- [ ] Research which `minio/console` tag or commit still contained the full CE UI (pre–May 2025).
- [ ] Option A: Pin `go.mod` to that version, e.g. `github.com/minio/console v1.x.x` or a specific pseudo-version.
- [ ] Option B: Fork `minio/console`, revert the commit that stripped CE, then use a `replace` in `go.mod` (or a fork module path) so minio depends on the forked console.

### 1.2 Update minio to use full console

- [ ] In `go.mod`: either change the `require` for `github.com/minio/console` to the chosen version, or add a `replace` to your console fork.
- [ ] Run `go mod tidy` and fix any breaking API changes between current and chosen console version (if any).
- [x] Document the choice in `README.md` or `docs/CONSOLE.md` — see [docs/CONSOLE.md](CONSOLE.md).

### 1.3 Verify console in CI

- [ ] Add a minimal CI check that the server starts and the console route responds (e.g. in `go.yml` or a dedicated workflow), so regressions are caught.

**Deliverable:** MinIO fork that serves the full admin console again; CI green.

---

## Phase 2: Rebuild Binary Distribution

**Objective:** Publish binaries and Docker images from this fork so users do not depend on `dl.min.io` or `minio/minio` images.

### 2.1 Build binaries from source

- [x] Use `buildscripts/gen-ldflags.go` in CI to produce release binaries for `linux/amd64`, `linux/arm64`.
- [x] Publish unsigned binaries with SHA256 checksums (no minisign in CI).
- [x] **Release workflow** `.github/workflows/release.yml`: triggers on tag `v*` or `RELEASE.*`, or manual with tag input; builds binaries; uploads to GitHub Releases.

### 2.2 Docker image from source

- [ ] Introduce a **build-from-source** Dockerfile (or adjust existing one) that:
  - Uses multi-stage build: stage 1 compiles minio from source (no download from dl.min.io).
  - Stage 2 copies the built binary into a minimal runtime image (e.g. `Dockerfile.release`-style base).
- [ ] Do **not** rely on `Dockerfile.release`’s `curl` from `dl.min.io`; remove or replace that with the new build-from-source flow.

### 2.3 Publish Docker images

- [ ] In the same release workflow (or a separate `docker-publish.yml`):
  - Build the new Docker image for `linux/amd64` and `linux/arm64` (and others if desired).
  - Push to a fork-owned registry:
    - **GitHub Container Registry:** `ghcr.io/prakash-sucify/minio`
    - Optionally **Docker Hub:** `prakash-sucify/minio` (requires Docker Hub credentials in secrets).
- [ ] Document in README: “Docker: `docker pull ghcr.io/prakash-sucify/minio:tag`”.

### 2.4 Optional: RPM/DEB packages

- [ ] If desired, add a job in the release workflow (or a separate workflow) to build RPM/DEB using the same binaries, and attach them to GitHub Releases or publish to a repo. (Lower priority than binaries + Docker.)

**Deliverable:** Every release produces GitHub Release artifacts (binaries + checksums) and Docker images under the fork’s namespace; no dependency on MinIO’s distribution.

---

## Phase 3: CVE and Bug Maintenance

**Objective:** Keep the fork safe and stable without adding features.

### 3.1 CVE tracking

- [ ] Keep `govulncheck` in CI (already in `vulncheck.yml`); fix or document any findings.
- [ ] Subscribe to Go security advisories and minio/console (or your console fork) for relevant CVEs.
- [ ] For each CVE affecting this codebase: patch, tag a patch release, and rebuild binaries + Docker (reuse Phase 2 workflows).

### 3.2 Bug fixes only

- [ ] Policy: accept only bug fixes and security patches; no new features (align with “supply chain continuity”).
- [ ] Document this in `README.md` and `CONTRIBUTING.md` (e.g. “This fork is maintained for stability and security only”).

### 3.3 Issue and release process

- [ ] Use GitHub Issues for bugs and security reports; close “feature request” issues with a pointer to this policy.
- [ ] Define a simple release cadence (e.g. patch releases when CVEs or critical bugs are fixed; optional periodic sync from upstream with minimal patches).

**Deliverable:** Clear maintenance policy; CVE handling process; CI that blocks on known vulnerabilities where feasible.

---

## Phase 4: Documentation (Community Edition)

**Objective:** Ensure users can install, run, and operate the fork without relying on broken or redirected upstream docs.

### 4.1 In-repo docs

- [ ] Review `docs/` in this repo; fix broken links and references to minio.com or deprecated endpoints.
- [ ] Add a short “Fork overview” in `README.md`: what this fork is, how it differs (console, distribution), and where to get binaries/Docker (links to GitHub Releases and image registry).

### 4.2 Optional: Fork minio/docs

- [ ] If you need a full doc site (e.g. for console usage, IAM, replication): fork `minio/docs`, fix redirects and removed console docs, and deploy to GitHub Pages or another host. Link from this repo’s README.
- [ ] If not: keep everything in-repo and link to upstream docs where still valid, with a disclaimer that this fork is independent.

**Deliverable:** README and in-repo docs accurate and self-contained; optional forked doc site if needed.

---

## Phase 5: Legal and Trademark

**Objective:** Reduce risk and set clear expectations.

### 5.1 Trademark disclaimer

- [ ] Add a **trademark disclaimer** in README and, if you have a website, on the site: this project is an independent, community-maintained fork under AGPL; not affiliated with MinIO, Inc.; “MinIO” is a registered trademark of MinIO, Inc.
- [ ] (Optional) Plan a neutral name (e.g. “sucify-minio” or “minio-ce”) if you ever need to rename to avoid trademark issues.

### 5.2 License and compliance

- [x] LICENSE and NOTICE unchanged from upstream (AGPL v3). New files (Dockerfile.build, workflows, docs) are consistent.
- [x] README states AGPL v3 and points to this repo for source.

**Deliverable:** Clear disclaimer and license information in repo and any published artifacts.

---

## Summary Table

| Phase | Focus | Priority |
|-------|--------|----------|
| **1** | Restore full admin console (pin or fork console) | High |
| **2** | Build and publish binaries + Docker from source | High |
| **3** | CVE tracking, bug-only maintenance, release process | High |
| **4** | Fix in-repo docs; optional minio/docs fork | Medium |
| **5** | Trademark disclaimer and license clarity | Medium |

---

## References

- [MinIO Is Dead, Long Live MinIO](https://blog.vonng.com/en/db/minio-resurrect/) — Vonng’s resurrection approach (console revert, binaries, Docker, docs).
- Upstream minio/minio (archived): https://github.com/minio/minio  
- MinIO console: https://github.com/minio/console  
- This fork: `origin` → `prakash-sucify/minio`

---

*This roadmap is a living document. Update it as phases are completed or priorities change.*

**→ To trigger your first release and run through the checklist:** [NEXT-STEPS.md](NEXT-STEPS.md)
