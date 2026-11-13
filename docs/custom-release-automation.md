# Custom release automation

This fork stores the upload defaults customization as a patch that is reapplied to upstream Gokapi releases.

## Workflow

The automation is in [custom-release.yml](../.github/workflows/custom-release.yml).

- It runs daily at 03:17 UTC and checks the latest stable release of Forceu/Gokapi.
- Manual runs accept a stable upstream tag, or `master` for validation without publishing.
- A push to `automation/custom-release-build` validates the patch against upstream master without publishing.
- Versions earlier than `v2.2.5` are skipped because the patch targets upstream 2.2.5-dev. The latest upstream release when this workflow was created was `v2.2.4`.
- Published version tags are checked before rebuilding.
- The workflow clones upstream, applies `patches/upload-defaults.patch`, runs `go generate ./...` and Go tests, then compiles Linux AMD64 and ARM64 binaries.
- Docker Buildx publishes a multi-platform image to GitHub Container Registry only after successful checks.
- Failed release builds stop before publication and attempt to create a GitHub issue. An existing open issue for the same version is reused.

Image tags for an upstream `v2.2.5` release:

- `ghcr.io/erathaowl/gokapi:v2.2.5-custom`
- `ghcr.io/erathaowl/gokapi:latest-custom` (only when v2.2.5 is the latest upstream release)

GitHub Actions uses `GITHUB_TOKEN` with `packages: write` and `issues: write`. No external registry credentials are needed.
Set the resulting GHCR package to public in GitHub package settings if unauthenticated pulls are required.

## Local verification

From a full Git checkout of the relevant upstream release, apply the patch and run the tests:

```bash
git apply --3way /path/to/Gokapi/patches/upload-defaults.patch
go generate ./...
go test ./... -parallel 8 -count=1 -tags=test,noaws
```

If an upstream change conflicts with the patch, the workflow fails and no image is published.

## Branch separation

Scheduled GitHub Actions workflows must be on the default branch. While this workflow remains on the automation branch, push events can validate it, but daily checks will start only after merging into `master`.

The automation is separate from the upstream contribution [Forceu/Gokapi#451](https://github.com/Forceu/Gokapi/pull/451), which currently uses `erathaowl:master` as its head. Merging this workflow into the fork's `master` while that PR is open would add unrelated workflow files to the upstream PR.

Configuration variables remain `GOKAPI_DEFAULT_DOWNLOADS` (default `1`) and `GOKAPI_DEFAULT_EXPIRY_DAYS` (default `14`).
