# `publish.yaml` — GitHub Actions workflow for publishing container images

`.github/workflows/publish.yaml` is a GitHub Actions workflow that builds and publishes multi‑architecture container images with Docker Buildx and QEMU.  
It is meant to be **called from another workflow file** in the _consuming_ repository.

---

## Quick start

Create a workflow in your project (e.g. `.github/workflows/publish-image.yml`) and call the workflow from this repository:

```yaml
jobs:
  publish:
    uses: MattKobayashi/actions-workflows@v1.1.0
    with:
      image-name: my‑app        # REQUIRED — name of the image on the registry
      dockerfile-path: ./       # OPTIONAL — folder that contains the Dockerfile
      artifact-name: build      # OPTIONAL — artifact with repo contents
      platforms: linux/amd64    # OPTIIONAL - comma-separated list of platforms to build the image for
      registry-url: ghcr.io     # OPTIONAL — registry URL
      registry-org: my‑org      # OPTIONAL — org/user on registry
    secrets: inherit            # Gives the called workflow access to `GITHUB_TOKEN`
name: Publish
on:
  release:
    # Only build a new image alongside published releases
    types:
      - published
```

---

## Input parameters

| Name            | Required | Default                    | Description                                                                                          |
| --------------- | -------- | -------------------------- | ---------------------------------------------------------------------------------------------------- |
| `image-name`    | ✔︎       | —                          | Name of the image in the container registry (`<registry-url>/<registry-org>/<image-name>`).                    |
| `dockerfile-path` |         | `GITHUB_WORKSPACE` | Path (file or directory) that is passed as the Buildx context.                                       |
| `artifact-name` |          | `""` | Name of an already‑uploaded artifact whose contents will be used. If this is not set, `git checkout` is used instead. |
| `platforms` |       | `linux/amd64` | A comma-separated list of platforms to build images for (e.g. `linux/amd64,linux/arm64`)       |
| `registry-url`  |          | `ghcr.io`                  | Registry host to log in to (e.g. `ghcr.io`, `docker.io`).                                            |
| `registry-org`  |          | `GITHUB_REPOSITORY_OWNER` (lower-case) | Organisation / user namespace on the registry (must be lower‑case for `ghcr.io`).                         |

---

## Tagging strategy

The workflow automatically generates and pushes three semver‑style tags for each git tag:

* `v<major>.<minor>.<patch>` (exact tag that triggered the run)
* `v<major>.<minor>`
* `v<major>`

Example: creating the git tag `v2.7.3` results in image tags `v2.7.3`, `v2.7`, and `v2`.

---

## What the workflow does

1. Gets the source code: either checks out the triggering repository or, if `artifact‑name` is set, downloads the artifact and extracts it to `GITHUB_WORKSPACE`.  
2. Sets up QEMU (for cross‑platform builds) and Docker Buildx.  
3. Logs in to the registry using `GITHUB_TOKEN`.  
4. Builds the image for all enabled Buildx platforms and pushes it to the registry.  
5. Applies OCI labels and the semver tags described above.

---

## Requirements & notes

* The caller must supply `GITHUB_TOKEN` (using `secrets: inherit` is easiest).

---

Happy publishing! 🎉
