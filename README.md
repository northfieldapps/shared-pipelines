# Shared Pipelines

Reusable GitHub Actions workflows for the organization. Call these from any repo using `uses: <org>/shared-pipelines/.github/workflows/<workflow>.yml@main`.

> **Note:** This repo must be **public** unless the org is on a GitHub Teams or Enterprise plan.

## Workflows

### `java-maven-build.yml` — Java/Maven Build & Test

Build and test any Maven-based Java project (Spring Boot, etc.).

| Input | Type | Default | Description |
|---|---|---|---|
| `java-version` | string | `'21'` | JDK version |
| `java-distribution` | string | `'temurin'` | JDK distribution |
| `maven-args` | string | `'-B verify'` | Maven command arguments |
| `working-directory` | string | `'.'` | Subdirectory with pom.xml |

```yaml
jobs:
  build:
    uses: <org>/shared-pipelines/.github/workflows/java-maven-build.yml@main
    with:
      java-version: '21'
```

---

### `node-build.yml` — Node.js/TypeScript Build & Test

Build and test any npm/pnpm/yarn project (Electron, browser extensions, etc.).

| Input | Type | Default | Description |
|---|---|---|---|
| `node-version` | string | `'20'` | Node.js version |
| `package-manager` | string | `'npm'` | `npm`, `pnpm`, or `yarn` |
| `build-script` | string | `'build'` | npm script for build |
| `test-script` | string | `'test'` | npm script for tests |
| `skip-build` | boolean | `false` | Skip the build step |
| `test-args` | string | `''` | Additional arguments for the test command |
| `coverage-upload` | boolean | `false` | Upload coverage report via Codecov |
| `working-directory` | string | `'.'` | Project subdirectory |
| `artifact-path` | string | `''` | Path to upload as artifact |
| `artifact-name` | string | `'build-output'` | Artifact name |

```yaml
jobs:
  build:
    uses: <org>/shared-pipelines/.github/workflows/node-build.yml@main
    with:
      node-version: '20'
      package-manager: pnpm
```

---

### `xcode-build.yml` — Xcode Build & Test (macOS/iOS)

Build and test Swift/SwiftUI apps on macOS runners.

| Input | Type | Default | Description |
|---|---|---|---|
| `scheme` | string | *required* | Xcode scheme to build |
| `project-path` | string | `'.'` | Path to `.xcodeproj` or `.xcworkspace` |
| `platform` | string | `'iOS Simulator'` | Destination platform |
| `os-version` | string | `'latest'` | Simulator OS version |
| `device` | string | `'iPhone 16'` | Simulator device |
| `xcode-version` | string | `'latest-stable'` | Xcode version |

```yaml
jobs:
  build:
    uses: <org>/shared-pipelines/.github/workflows/xcode-build.yml@main
    with:
      scheme: MyApp
      platform: 'iOS Simulator'
```

---

### `browser-extension-build.yml` — Chrome/Firefox Extension Packaging

Build and package browser extensions for Chrome and Firefox.

| Input | Type | Default | Description |
|---|---|---|---|
| `node-version` | string | `'20'` | Node.js version |
| `build-script` | string | `'build'` | npm build script |
| `chrome-manifest` | string | `''` | Path to Chrome-specific manifest |
| `skip-npm-build` | boolean | `false` | Skip Node setup, npm install, and npm build |
| `chrome-build-script` | string | `''` | Path to shell script for Chrome build |
| `firefox-build-script` | string | `''` | Path to shell script for Firefox build |
| `chrome-artifact` | string | `'chrome-extension.zip'` | Chrome output zip filename |
| `firefox-artifact` | string | `'firefox-extension.zip'` | Firefox output zip filename |
| `artifact-name` | string | `'extension'` | Artifact name |

```yaml
jobs:
  build:
    uses: <org>/shared-pipelines/.github/workflows/browser-extension-build.yml@main
    with:
      build-script: 'build:prod'
```

---

### `docker-build-push.yml` — Build & Push Docker Image

Build and push Docker images to GHCR or Docker Hub with multi-platform support.

| Input | Type | Default | Description |
|---|---|---|---|
| `image-name` | string | *required* | Image name |
| `registry` | string | `'ghcr.io'` | `ghcr.io` or `docker.io` |
| `dockerfile` | string | `'Dockerfile'` | Path to Dockerfile |
| `context` | string | `'.'` | Build context |
| `platforms` | string | `'linux/amd64'` | Target platforms |
| `push` | boolean | `true` | Push or build-only |
| `tags` | string | `''` | Additional tags |

| Secret | Description |
|---|---|
| `DOCKERHUB_USERNAME` | Docker Hub username (when using docker.io) |
| `DOCKERHUB_TOKEN` | Docker Hub access token |

```yaml
jobs:
  docker:
    uses: <org>/shared-pipelines/.github/workflows/docker-build-push.yml@main
    with:
      image-name: my-app
      registry: ghcr.io
    secrets: inherit
```

---

### `github-release.yml` — Create GitHub Release

Create a GitHub release with auto-generated notes and uploaded artifacts.

| Input | Type | Default | Description |
|---|---|---|---|
| `tag` | string | *required* | Git tag (e.g. `v1.2.3`) |
| `artifact-name` | string | `''` | Artifact to download from prior job |
| `artifact-path` | string | `'dist/'` | Files to attach to release |
| `release-notes` | string | `''` | Custom release notes body text |
| `generate-notes` | boolean | `true` | Auto-generate release notes (ignored when `release-notes` is set) |
| `draft` | boolean | `false` | Create as draft |
| `prerelease` | boolean | `false` | Mark as prerelease |

```yaml
jobs:
  release:
    if: startsWith(github.ref, 'refs/tags/v')
    uses: <org>/shared-pipelines/.github/workflows/github-release.yml@main
    with:
      tag: ${{ github.ref_name }}
    permissions:
      contents: write
```

---

## Full Caller Example

A typical CI pipeline combining build, Docker, and release:

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    uses: <org>/shared-pipelines/.github/workflows/java-maven-build.yml@main
    with:
      java-version: '21'

  docker:
    needs: build
    uses: <org>/shared-pipelines/.github/workflows/docker-build-push.yml@main
    with:
      image-name: my-spring-app
      registry: ghcr.io
    secrets: inherit

  release:
    if: startsWith(github.ref, 'refs/tags/v')
    needs: build
    uses: <org>/shared-pipelines/.github/workflows/github-release.yml@main
    with:
      tag: ${{ github.ref_name }}
    permissions:
      contents: write
```
