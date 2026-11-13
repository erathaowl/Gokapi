# Automated custom Gokapi releases

The workflow in `.github/workflows/build-patched-release.yml` applies
`patches/upload-defaults.patch` to official Gokapi releases and publishes a
custom multi-platform image on GHCR.

## Usage

1. Merge this workflow into the fork's default branch and enable GitHub Actions.
2. Open **Actions > Build patched Gokapi releases > Run workflow**.
3. Choose `upstream_ref: master` and leave `dry_run` enabled for an initial
   validation against upstream development.
4. For a released tag, set `upstream_ref` to its version and disable `dry_run`
   to publish. Leave the version blank to use GitHub's latest published release.

The workflow checks for new releases daily at 04:17 UTC. Previously published
version tags are skipped, and `master` is always build-only.

The image name is:

```text
ghcr.io/erathaowl/gokapi:vX.Y.Z-custom
```

Deploy it with the same volumes and ports as the official image. The optional
upload defaults remain configurable:

```yaml
environment:
  GOKAPI_DEFAULT_DOWNLOADS: "10"
  GOKAPI_DEFAULT_EXPIRY_DAYS: "30"
```

GHCR packages may initially be private; change the package visibility in the
GitHub package settings if anonymous image pulls are required.

## Compatibility and failures

The patch is based on upstream commit
`34b54832a115f9a0bea495253f81d69d5c23bfe9`. Releases before this commit
are skipped. Each eligible release is checked out with its complete history
so that `git apply --3way` can use the original Git blobs to adapt the patch.

Generated/minified JavaScript is intentionally excluded from the patch and
recreated with `go generate ./...`. Go tests run before the Docker build.
A failed patch, test, or build blocks publication. GitHub Actions creates an
issue on the first build failure while an existing open failure issue prevents
duplicates.

If an upstream ref changes the patched code incompatibly, update
`patches/upload-defaults.patch` and re-run the workflow manually. Avoid
editing the upstream checkout in the workflow; maintain the custom changes in
the patch file.

## Image build

The upstream Dockerfile is used with Buildx and QEMU to produce
`linux/amd64` and `linux/arm64` images. Published tags correspond to upstream
release versions; dry-run builds are not uploaded.
