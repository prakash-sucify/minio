# Next Steps After Fork Setup

Use this checklist to ship your first release and optional improvements.

---

## 1. Commit and push your changes

Ensure all resurrection changes are committed and pushed to your fork:

```bash
cd /path/to/minio
git status
git add Dockerfile.build .github/workflows/release.yml docs/ README.md CONTRIBUTING.md
git commit -m "chore: add build-from-source, release workflow, and fork docs"
git push origin master
```

(Adjust branch name if you use `main` or another branch.)

---

## 2. Trigger the first release

The **Release (build from source)** workflow builds binaries and Docker images and creates a GitHub Release.

**Option A – Push a tag (recommended)**

```bash
# Use a release-style tag (timestamp or semver)
git tag RELEASE.2026-03-11T12-00-00Z
git push origin RELEASE.2026-03-11T12-00-00Z
```

Or a semantic tag:

```bash
git tag v1.0.0-fork
git push origin v1.0.0-fork
```

**Option B – Run the workflow manually**

1. Open your fork on GitHub: `https://github.com/prakash-sucify/minio`
2. Go to **Actions** → **Release (build from source)** → **Run workflow**
3. Enter a tag (e.g. `RELEASE.2026-03-11T12-00-00Z`) and run.

**After the run**

- **GitHub Releases:** Binaries and SHA256 checksums for linux/amd64 and linux/arm64.
- **GHCR:** Image `ghcr.io/prakash-sucify/minio:<tag>` and `ghcr.io/prakash-sucify/minio:latest`.

Test the image:

```bash
docker pull ghcr.io/prakash-sucify/minio:latest
docker run -p 9000:9000 -p 9001:9001 ghcr.io/prakash-sucify/minio server /data --console-address :9001
```

---

## 3. (Optional) Restore full admin console

To get the pre–May 2025 full admin UI (user management, bucket policies, etc.):

```bash
go get github.com/minio/console@v1.7.6
go mod tidy
make verify
make test-timeout
```

If the build or tests fail, see [CONSOLE.md](CONSOLE.md) for forking the console and using a `replace` in `go.mod`. If they pass, commit and push, then cut a new release (step 2).

---

## 4. Fix origin remote (security)

If `origin` was set with a URL that contains a GitHub token, switch to HTTPS without the token or to SSH:

```bash
# Use HTTPS (Git will prompt or use credential helper)
git remote set-url origin https://github.com/prakash-sucify/minio.git

# Or use SSH
git remote set-url origin git@github.com:prakash-sucify/minio.git
```

Then **revoke the old token** in GitHub: Settings → Developer settings → Personal access tokens.

---

## 5. Ongoing maintenance

- **CVEs:** Rely on `govulncheck` in CI (`.github/workflows/vulncheck.yml`). Fix or document findings and cut a patch release.
- **Policy:** Only bug fixes and security patches; no new features (see [CONTRIBUTING.md](../CONTRIBUTING.md)).
- **Releases:** Tag or run the release workflow when you have a stable build to publish.

See [ROADMAP-RESURRECTION.md](ROADMAP-RESURRECTION.md) for the full plan and status.
