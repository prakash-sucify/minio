# Admin Console (Community Edition)

## Background

As of **May 2025**, the upstream MinIO Community Edition removed the full admin console from the default build. The web UI was reduced to a basic object browser; user management, bucket policies, access control, and lifecycle management were removed from the community build.

This fork aims to keep the **full admin console** available for community users when possible.

## Current state

The server currently depends on `github.com/minio/console` as a Go module. The exact version is pinned in the repository root `go.mod`.

- **To restore the full CE admin console**, pin the console dependency to a version from **before May 2025** (e.g. a tag that still included the full UI).
- Upstream references: [minio/console](https://github.com/minio/console), [minio/object-browser](https://github.com/minio/object-browser) (later rename).

## Restoring the full console (optional)

If you want to use a pre–May 2025 console with full admin features:

1. In the repo root, run:
   ```bash
   go get github.com/minio/console@v1.7.6
   go mod tidy
   ```
2. If the build or API no longer matches (e.g. breaking changes in the server), you may need to:
   - Use a different console tag/commit, or
   - Maintain a small fork of `minio/console` that restores the full UI and keep this repo’s `go.mod` pointing at that fork (e.g. via `replace` or a fork module path).

After updating the console version, run `make verify` and the test suite to confirm everything compiles and passes.

## References

- [Critical WEB UI features removed in Community Edition (minio/minio #21584)](https://github.com/minio/minio/issues/21584)
- [MinIO Is Dead, Long Live MinIO (Vonng)](https://blog.vonng.com/en/db/minio-resurrect/) — community approach to restoring the console and supply chain
