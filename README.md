# Gokapi (custom fork)

[![Documentation Status](https://readthedocs.org/projects/gokapi/badge/?version=latest)](https://gokapi.readthedocs.io/en/stable/?badge=stable)
[![Go Report Card](https://goreportcard.com/badge/github.com/forceu/gokapi)](https://goreportcard.com/report/github.com/forceu/gokapi)
[![Coverage](https://img.shields.io/badge/Go%20Coverage-83%25-brightgreen.svg?longCache=true&style=flat)](https://github.com/jpoles1/gopherbadger)
[![Custom image build](https://github.com/erathaowl/Gokapi/actions/workflows/custom-release.yml/badge.svg)](https://github.com/erathaowl/Gokapi/actions/workflows/custom-release.yml)

This is a customized fork of [Forceu/Gokapi](https://github.com/Forceu/Gokapi), adding configurable default upload limits and automated Docker images in [GitHub Container Registry](https://github.com/erathaowl/Gokapi/pkgs/container/gokapi).

**Gokapi** is a modern, self-hosted alternative to Firefox Send that puts you in control of your file sharing. Built with Go, it combines simplicity with powerful features like automatic file expiration, end-to-end encryption, and flexible cloud storage support.

### Available for:

- **Bare Metal** (Linux/macOS/Windows)
- **Docker (this fork)**: `ghcr.io/erathaowl/gokapi` ([image builds](https://github.com/erathaowl/Gokapi/actions/workflows/custom-release.yml))
- **Official upstream Docker image**: [f0rc3/gokapi on Docker Hub](https://hub.docker.com/r/f0rc3/gokapi)

## Features

- **Expiring file shares:** Automatically removed after a set number of downloads or days
- **User management with roles:** Fine-grained permission control, only registered users can upload
- **File requests:** A shareable URL lets external parties upload files, visible only to the URL’s creator
- **File deduplication:** Identical files use no extra space
- **Cloud storage support:** AWS S3 (or S3 compatible like Backblaze B2), optional
- **Built-in encryption:** Including end-to-end encrypted uploads
- **OpenID Connect support:** Integrate with identity providers like Authelia or Keycloak
- **REST API:** For automation and integration into other systems
- **Customizable UI:** Adjust look and feel with custom CSS and JavaScript

---

## Configurable upload defaults

Set these optional environment variables to change the defaults for new uploads:

```yaml
environment:
  GOKAPI_DEFAULT_DOWNLOADS: "10"
  GOKAPI_DEFAULT_EXPIRY_DAYS: "30"
```

The defaults are 1 download and 14 days when the variables are unset. Set either variable to `0` for an unlimited number of downloads or no expiration. These values apply to uploads through the web interface and API requests that omit the corresponding parameter. Explicit upload limits take precedence, and the web interface keeps any existing preferences stored in the browser.

## Custom Docker images

This fork builds Gokapi from [upstream](https://github.com/Forceu/Gokapi), applies the upload-default patch, and publishes Linux AMD64/ARM64 images to GHCR.

| Tag | Source | Updated |
| --- | --- | --- |
| `vX.Y.Z` | Exact upstream release | Once, when a compatible release is available |
| `upstream` | Latest compatible upstream release | Automatically on new upstream releases |
| `latest` | Same image as `upstream` | Automatically on new upstream releases |
| `dev` | Current upstream `master` on manual builds, or the merged fork `master` commit after a PR merge | Manual workflow runs and merges into fork `master` |
| `master` | Same image as `dev` | Only when enabled during a manual workflow run |

The `dev` and optional `master` tags are development builds. Stable release images start with upstream `v2.2.5`, since the patch targets Gokapi 2.2.5-dev. GHCR tags become available after their first successful build and publication.

Each successfully published stable version also gets a [GitHub Release](https://github.com/erathaowl/Gokapi/releases) named `vX.Y.Z-custom`. Its source tag references a patched snapshot of that upstream release, and the release notes record the matching Docker image digest. Development builds do not create GitHub Releases.

Merging a pull request into this fork's `master` automatically rebuilds and publishes `dev` from the merged commit. This event does not update the optional `master` tag. Closing an unmerged PR does not publish an image.

To build and publish from upstream `master`, open [Build custom Gokapi images](https://github.com/erathaowl/Gokapi/actions/workflows/custom-release.yml), select **Run workflow**, set **upstream_ref** to `master`, and optionally enable **tag_master**. Leave `upstream_ref` empty to check the latest upstream release.

The GitHub Container Registry package may need to be [made public](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility) to allow unauthenticated pulls. See [automation details](docs/custom-release-automation.md).

## Getting Started

You can deploy Gokapi in seconds using Docker or directly on your system.

[Installation Guide](https://gokapi.readthedocs.io/en/latest/setup.html)  
[Usage Instructions](https://gokapi.readthedocs.io/en/latest/usage.html)

**Want to give it a try?**

Start this customized Gokapi image with Docker after publishing the `dev` image:

```bash
docker run --rm \
  --name gokapi \
  -v gokapi-data:/app/data \
  -v gokapi-config:/app/config \
  -p 127.0.0.1:53842:53842 \
  -e TZ=UTC \
  ghcr.io/erathaowl/gokapi:dev
```

Then visit ``http://localhost:53842/setup`` and follow the setup wizard.



## Screenshots
**Main Menu**

<a href="https://github.com/user-attachments/assets/d805a88b-dc74-4c39-bed6-ec31b9c3e17f" target="_blank">
  <img width="300" alt="image" src="https://github.com/user-attachments/assets/d805a88b-dc74-4c39-bed6-ec31b9c3e17f" />

</a>

**File Requests**

<a href="https://github.com/user-attachments/assets/a6565cf8-bd2d-4027-a150-673aa93d4502" target="_blank">
 <img width="300"  alt="image" src="https://github.com/user-attachments/assets/a6565cf8-bd2d-4027-a150-673aa93d4502" />
</a>



**User Overview**

<a href="https://github.com/user-attachments/assets/cbc738e4-75ae-4647-8178-da735f74a86f" target="_blank">
  <img width="300" alt="image" src="https://github.com/user-attachments/assets/cbc738e4-75ae-4647-8178-da735f74a86f" />
</a>


**API Overview**

<a href="https://github.com/user-attachments/assets/c480af8e-772c-4f8b-9f0e-28c8aceb9b49" target="_blank">
  <img width="300" alt="image" src="https://github.com/user-attachments/assets/c480af8e-772c-4f8b-9f0e-28c8aceb9b49" />
</a>

**Status Overview**

<a href="https://github.com/user-attachments/assets/70d5ab07-e60f-48d5-8739-fa038129e5ae" target="_blank">
<img width="300" alt="image" src="https://github.com/user-attachments/assets/70d5ab07-e60f-48d5-8739-fa038129e5ae" />
</a>


**Download Link**

<a href="https://github.com/user-attachments/assets/fd9c032b-733d-4657-9f42-f751b2634e02" target="_blank">
  <img width="300" alt="image" src="https://github.com/user-attachments/assets/fd9c032b-733d-4657-9f42-f751b2634e02" />

</a>

---

## System Requirements

### Minimum
- **CPU**: 1 core
- **RAM**: 256 MB
- **Storage**: 100 MB + file storage
- **OS**: Linux, macOS, Windows

### Recommended  
- **CPU**: 2+ cores
- **RAM**: 512 MB+
- **Storage**: SSD strongly recommended

## Development

### Building from Source

```bash
# Clone the repository
git clone https://github.com/erathaowl/Gokapi.git
cd gokapi

# Build the binary
make build

# Run tests
make test

# Build Docker image
docker build -t gokapi:local .
```

## License

This project is licensed under the AGPL3 - see the [LICENSE.md](LICENSE.md) file for details

## Contributors
<a href="https://github.com/forceu/gokapi/graphs/contributors">
  <img src="https://contributors-img.web.app/image?repo=forceu/gokapi" />
</a>



## Donations

As with all Free software, the power is less in the finances and more in the collective efforts. I really appreciate every pull request and bug report offered up by our users! If however, you're not one for coding/design/documentation, and would like to contribute financially, you can do so with the link below. Every help is very much appreciated!

[![paypal](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=donate@bulling.mobi&lc=US&item_name=BarcodeBuddy&no_note=0&cn=&currency_code=EUR&bn=PP-DonationsBF:btn_donateCC_LG.gif:NonHosted) [![LiberaPay](https://img.shields.io/badge/Donate-LiberaPay-green.svg)](https://liberapay.com/MBulling/donate)

Powered by [Jetbrains](https://jb.gg/OpenSourceSupport)





