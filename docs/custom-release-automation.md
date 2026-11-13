# Custom release automation

This fork keeps the upload defaults customization in `patches/upload-defaults.patch` and reapplies it to upstream Gokapi before compiling images.

## Workflow

See [Build custom Gokapi images](../.github/workflows/custom-release.yml) or [run it in GitHub Actions](https://github.com/erathaowl/Gokapi/actions/workflows/custom-release.yml).

- A schedule checks for new upstream stable releases every day at 03:17 UTC.
- `workflow_dispatch` supports `upstream_ref`: leave empty for the latest upstream release, enter a stable version such as `v2.2.5`, or enter `master` for a development image.
- Set the optional `tag_master` input to true when using `upstream_ref=master` to add the moving `master` Docker tag.
- Upstream stable versions older than `v2.2.5` are skipped because the patch targets upstream Gokapi 2.2.5-dev.
- Already-published stable version images are reused. The `upstream` and `latest` aliases are still updated when that version is the latest upstream release.
- A manual `master` build always re-generates and publishes the `dev` image, regardless of previous runs.
- When a pull request is merged into this fork's `master`, the workflow builds the merged commit from `erathaowl/Gokapi` and publishes `dev` automatically. It does not apply the patch again, because the source already contains the customization. Closing a PR without merging does not start a build.
- Automatic PR-merge builds update only `dev`. The optional `master` tag is controlled exclusively by the manual `tag_master` input.
- The workflow clones upstream, applies the patch with `git apply --3way`, generates assets, runs Go tests, builds AMD64 and ARM64 Linux binaries, then publishes a multi-platform image with Docker Buildx.
- If the patch, tests, binary compilation, or publication fail, the workflow stops and attempts to open a GitHub issue.

## Docker image tags

| Trigger | Image tags |
| --- | --- |
| New compatible upstream release `vX.Y.Z` | `ghcr.io/erathaowl/gokapi:vX.Y.Z`, `:upstream`, `:latest` |
| Manual run for an older supported version | `ghcr.io/erathaowl/gokapi:vX.Y.Z` only |
| Manual run with `upstream_ref=master` | `ghcr.io/erathaowl/gokapi:dev` |
| Manual run with `upstream_ref=master`, `tag_master=true` | `ghcr.io/erathaowl/gokapi:dev`, `:master` |
| Pull request merged into fork `master` | `ghcr.io/erathaowl/gokapi:dev` |

The mobile tags `upstream` and `latest` point to the most recently published compatible upstream release. A failed release build does not move them. Existing versioned images are not rebuilt by default.

## Authentication

GitHub Actions uses `GITHUB_TOKEN` with `packages: write` and `issues: write`; Docker Hub credentials are not needed. To allow pulls without authentication, make the GitHub Container Registry package public after its first publication.

## Local verification

From a clean checkout of the relevant upstream commit or tag, apply the patch and run the tests:

```bash
git apply --3way /path/to/Gokapi/patches/upload-defaults.patch
go generate ./...
go test ./... -parallel 8 -count=1 -tags=test,noaws
```

An upstream change can conflict with the patch. In that case the workflow fails without publishing a new image; update and revalidate `patches/upload-defaults.patch`.

## Upstream contribution

The upstream contribution is [Forceu/Gokapi#452](https://github.com/Forceu/Gokapi/pull/452) from `erathaowl:feature/upload-defaults-upstream`. Automation and README changes remain on the fork's `master` without changing that PR.

Available settings: `GOKAPI_DEFAULT_DOWNLOADS` (default `1`) and `GOKAPI_DEFAULT_EXPIRY_DAYS` (default `14`), with `0` meaning unlimited.
