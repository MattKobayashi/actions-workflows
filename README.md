# Reusable GitHub Actions workflows

## `test.yaml` - GitHub Actions workflow for executing Compose-defined tests

`.github/workflows/test.yaml` is a GitHub Actions workflow that executes Compose-defined tests. It also performs a Docker Scout analysis on the built image to detect CVEs and make general recommendations.

---

### Quick start

Create a workflow in your project (e.g. `.github/workflows/test-image.yml`) and call the workflow from this repository:

```yaml
jobs:
  test:
    uses: MattKobayashi/actions-workflows/.github/workflows/test.yaml@6770ac44f4019b59c3da45c47702d0077354027f # v1.3.0
    with:
      app-container-name: app
      test-container-name: connection_test
      test-directory: tests
name: Test
on:
  pull_request:
    # Only test PRs targeting the main branch
    branches:
      - main
  workflow_dispatch:
permissions:
  contents: read
  pull-requests: write
```

---

### Input parameters

| Name                  | Required | Default           | Description                                                                                                                                                                                                       |
| --------------------- | -------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `app-container-name`  | ✘        | `app`             | Name of the container running the application under test, as defined in the Compose file.                                                                                                                         |
| `test-container-name` | ✘        | `connection_test` | Name of the container running the tests (connection, expected output, etc.), as defined in the Compose file. This can be the same as `app-container-name` if your tests are run in the same container as the app. |
| `test-directory`      | ✘        | `tests`           | Directory containing your test Compose file.                                                                                                                                                                      |

---

### Example `docker-compose.yaml` (simple connection test)

```yaml
---
name: test
services:
  ## Connection Test ##
  connection_test:
    command: >
      bash -c "apt-get update
      && apt-get install --no-install-recommends --yes netcat-openbsd
      && nc -zv app 3000"
    container_name: connection_test
    depends_on:
      - app
    image: debian:trixie-slim
    networks:
      - test
    restart: "no"
  ## App ##
  app:
    build:
      context: ..
      dockerfile: Dockerfile
    container_name: app
    networks:
      - test
    restart: unless-stopped
networks:
  test:
    name: test
    driver: bridge
    enable_ipv6: true
    ipam:
      config:
        - subnet: 10.42.0.0/24
        - subnet: fdea:420:cafe::/64
```

---

## `publish.yaml` — GitHub Actions workflow for publishing container images

`.github/workflows/publish.yaml` is a GitHub Actions workflow that builds and publishes multi‑architecture container images with Docker Buildx and QEMU.  
It is meant to be **called from another workflow file** in the _consuming_ repository.

---

### Quick start

Create a workflow in your project (e.g. `.github/workflows/publish-image.yml`) and call the workflow from this repository:

```yaml
jobs:
  publish:
    name: Publish
    secrets: inherit
    uses: MattKobayashi/actions-workflows/.github/workflows/publish.yaml@6770ac44f4019b59c3da45c47702d0077354027f # v1.3.0
    with:
      image-name: my‑app
      dockerfile-path: ./
      artifact-name: build
      platforms: linux/amd64
      registry-url: ghcr.io
      registry-org: my‑org
    secrets: inherit # Gives the called workflow access to `GITHUB_TOKEN`
name: Publish
on:
  release:
    # Only build a new image alongside published releases
    types:
      - published
permissions:
  contents: read
  packages: write
```

---

### Input parameters

| Name              | Required | Default                                | Description                                                                                                           |
| ----------------- | -------- | -------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `image-name`      | ✔︎        | —                                      | Name of the image in the container registry (`<registry-url>/<registry-org>/<image-name>`).                           |
| `dockerfile-path` | ✘        | `GITHUB_WORKSPACE`                     | Path (file or directory) that is passed as the Buildx context.                                                        |
| `artifact-name`   | ✘        | `""`                                   | Name of an already‑uploaded artifact whose contents will be used. If this is not set, `git checkout` is used instead. |
| `platforms`       | ✘        | `linux/amd64`                          | A comma-separated list of platforms to build images for (e.g. `linux/amd64,linux/arm64`)                              |
| `registry-url`    | ✘        | `ghcr.io`                              | Registry host to log in to (e.g. `ghcr.io`, `docker.io`).                                                             |
| `registry-org`    | ✘        | `GITHUB_REPOSITORY_OWNER` (lower-case) | Organisation / user namespace on the registry (must be lower‑case for `ghcr.io`).                                     |

---

### Tagging strategy

The workflow automatically generates and pushes three semver‑style tags for each git tag:

- `v<major>.<minor>.<patch>` (exact tag that triggered the run)
- `v<major>.<minor>`
- `v<major>`

Example: creating the git tag `v2.7.3` results in image tags `v2.7.3`, `v2.7`, and `v2`.

---

### What the workflow does

1. Gets the source code: either checks out the triggering repository or, if `artifact‑name` is set, downloads the artifact and extracts it to `GITHUB_WORKSPACE`.
2. Sets up QEMU (for cross‑platform builds) and Docker Buildx.
3. Logs in to the registry using `GITHUB_TOKEN`.
4. Builds the image for all enabled Buildx platforms and pushes it to the registry.
5. Applies OCI labels and the semver tags described above.

---

### Requirements & notes

- The caller must supply `GITHUB_TOKEN` (using `secrets: inherit` is easiest).

---
