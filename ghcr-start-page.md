# GHCR (ghcr.io) — Start Page

This page helps you get started with GitHub Container Registry (ghcr.io): authenticate, build, push, pull, set visibility, and automate with GitHub Actions.

---

## What is GHCR?
GitHub Container Registry (ghcr.io) is GitHub's container image registry. Use it to publish and consume OCI images (containers) tied to users, repositories, or organizations. ghcr.io replaces the older docker.pkg.github.com and supports fine-grained permissions, visibility, and integration with GitHub Actions.

---

## Prerequisites
- Docker (or any OCI-compatible client)
- A GitHub account
- For local publishing: a Personal Access Token (PAT) with appropriate scopes (write:packages and read:packages; for private repos also repo)
- For GitHub Actions: the built-in GITHUB_TOKEN can be used for repo-scoped publish/read

---

## Authentication

### Local (using PAT)
Create a PAT with:
- For public packages: read:packages (to pull) and write:packages (to push)
- For private repositories: include `repo` as needed

Login:
```bash
echo $PAT | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

### GitHub Actions
Use the automatically provided `GITHUB_TOKEN`. Make sure job permissions include:
```yaml
permissions:
  packages: write
  contents: read
```
(You can adjust `packages: read` for pulling only.)

---

## Quickstart — Build, Tag, Push, Pull

Build and tag:
```bash
docker build -t ghcr.io/OWNER/IMAGE:TAG .
```

Push:
```bash
# Ensure you're logged in (see Authentication)
docker push ghcr.io/OWNER/IMAGE:TAG
```

Pull:
```bash
docker pull ghcr.io/OWNER/IMAGE:TAG
```

Notes:
- `OWNER` can be a user or an organization.
- For organization-scoped images, visibility and permissions follow org settings.

---

## Example: GitHub Actions — Build & Publish

Save this as `.github/workflows/publish-image.yml`:

```yaml
name: Build and publish container

on:
  push:
    branches: [ main ]    # adjust as needed
  workflow_dispatch:

jobs:
  build-and-publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to GHCR
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository_owner }}/my-image:${{ github.sha }}
          # add more tags if needed, e.g. latest, semver etc.
```

Tips:
- `secrets.GITHUB_TOKEN` works for the repository the workflow runs in without creating a PAT.
- For cross-repo or org-wide publishing you may need a PAT stored in repo secrets.

---

## Visibility & Access Control
- Published packages default to the visibility of the repository (public/private) unless changed.
- To make an image public, go to the package in the GitHub UI (Repository → Packages → your package) and change visibility.
- You can also manage package permissions for organizations and teams.

---

## Best Practices
- Use semantic version tags and immutable tags for production images (avoid reusing same tag for different contents).
- Tag images with both commit SHA and human-readable version (e.g., `v1.2.3` and `sha-<short>`).
- Scan images: use GitHub Container Scanning (Dependabot / GitHub Advanced Security) or other scanners in CI.
- Keep build context small and multi-stage builds to minimize image size.
- Use a CI job to clean up old images or use package retention policies to reduce storage.

---

## Cleanup / Delete
- You can delete package versions from the GitHub UI (Packages → version → Delete).
- For automation, use GitHub Packages API (check latest REST API docs). Deleting programmatically requires appropriate scopes.

---

## Troubleshooting
- 401/403: check token scopes and that you used `ghcr.io` as the registry and your username for login.
- 404 on pull: check image name, tag, and visibility (private images require authentication).
- Push fails: ensure package write permissions for the token (or GITHUB_TOKEN permissions in Actions).

---

## Useful Links
- ghcr.io (documentation): https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry
- Docker action: https://github.com/docker/build-push-action
- Docker login action: https://github.com/docker/login-action

---

## Example commands summary
```bash
# Login
echo $PAT | docker login ghcr.io -u YOUR_USERNAME --password-stdin

# Build and push
docker build -t ghcr.io/OWNER/IMAGE:TAG .
docker push ghcr.io/OWNER/IMAGE:TAG

# Pull
docker pull ghcr.io/OWNER/IMAGE:TAG
```

---

If you'd like, I can:
- Generate the `.github/workflows/publish-image.yml` workflow tailored to your repository (give me OWNER/REPO and desired image name/tag), or
- Create a small sample repo layout (Dockerfile + workflow) you can copy and run locally.
