# publish.yaml — Re‑usable GitHub Actions workflow for container images

This repository contains `.github/workflows/publish.yaml`, a _re‑usable_ GitHub Actions workflow that builds and publishes multi‑architecture container images with Docker Buildx and QEMU.  
It is meant to be **called from another workflow file** in the _consuming_ repository.

---

## Quick start

Create a workflow in your project (e.g. `.github/workflows/publish-image.yml`) and call the workflow from this repository:

```yaml
name: Publish image
on:
  push:
    # Only build/push when a git tag is created
    tags:
      - 'v*.*.*'      # e.g. v1.2.3

jobs:
  publish:
    uses: <OWNER>/<REPO>@<REF>  # e.g. my-org/reusable-publish/.github/workflows/publish.yaml@v1
    with:
      image-name: my‑app        # REQUIRED — name of the image on the registry
      dockerfile-path: ./       # OPTIONAL — folder that contains the Dockerfile (default: repo root)
      registry-url: ghcr.io     # OPTIONAL — registry host (default: ghcr.io)
      registry-org: my‑org      # OPTIONAL — org/user on registry (defaults to `${{ github.actor }}` lower‑case)
    secrets: inherit            # Gives the called workflow access to GITHUB_TOKEN
```

---

## Input parameters

| Name            | Required | Default                    | Description                                                                                          |
| --------------- | -------- | -------------------------- | ---------------------------------------------------------------------------------------------------- |
| `image-name`    | ✔︎       | —                          | Name of the image in the registry (`<registry-url>/<registry-org>/<image-name>`).                    |
| `dockerfile-path` |         | `${{ github.workspace }}` | Path (file or directory) that is passed as the Buildx context.                                       |
| `registry-url`  |          | `ghcr.io`                  | Registry host to log in to (e.g. `ghcr.io`, `docker.io`).                                            |
| `registry-org`  |          | `${{ github.actor }}` (lc) | Organisation / user namespace on the registry (must be lower‑case for GHCR).                         |

---

## Tagging strategy

The workflow automatically generates and pushes three semver‑style tags for each git tag:

* `v<major>.<minor>.<patch>` (exact tag that triggered the run)
* `v<major>.<minor>`
* `v<major>`

Example: creating the git tag `v2.7.3` results in image tags `v2.7.3`, `v2.7`, and `v2`.

---

## What the workflow does

1. Checks out the repository that **triggered** the workflow call.  
2. Sets up QEMU (for cross‑platform builds) and Docker Buildx.  
3. Logs in to the registry using `GITHUB_TOKEN`.  
4. Builds the image for all enabled Buildx platforms and pushes it to the registry.  
5. Applies OCI labels and the semver tags described above.

---

## Requirements & notes

* The caller must supply `GITHUB_TOKEN` (using `secrets: inherit` is easiest).  
* The action relies on git **tags**; if you want to build on every push, adapt the `on:` section accordingly.  
* The default Buildx builder is used; customise platforms or build args via a fork if needed.  

---

Happy publishing! 🎉
