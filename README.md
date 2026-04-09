# Custom Grafana Alloy Docker Image (GHCR)

This repository builds a custom Docker image based on the official [Grafana Alloy](https://hub.docker.com/r/grafana/alloy) image, injecting your own configuration, and pushes it to GitHub Container Registry (GHCR).

## How it works

- The Dockerfile uses the official Alloy image as a base and copies in your custom configuration.
- A GitHub Actions workflow checks for new releases of the official Alloy image (on push to main, on a schedule, or manually).
- If a new version is found, it automatically builds and pushes a new image with your configuration to GHCR.

## Workflow Triggers

The workflow runs on:
- Pushes to the `main` branch
- A daily schedule (6am UTC)
- Manual dispatch via the GitHub Actions UI

## How the Workflow Works

- **Fetch latest upstream tag:**
  - Uses a shell step with `curl` and `jq` to fetch the latest stable tag from Docker Hub for `grafana/alloy`.
- **Compare with last built tag:**
  - If the tag is new, proceeds to build and push.
- **Log in to GHCR:**
  - Uses `docker/login-action` to authenticate to GitHub Container Registry with the built-in `GITHUB_TOKEN`.
- **Build and push:**
  - Uses `docker/build-push-action` to build the image with the correct base tag and push to `ghcr.io/<owner>/alloy-custom:<tag>`.
- **Update last built tag:**
  - Stores the last built tag in `.last_built_tag` and commits it to the repository.

## Usage

1. Place your Alloy configuration in `alloy-config.yaml`.
2. The image will be built and published to GHCR as `ghcr.io/<your-org-or-username>/alloy-custom:<upstream-tag>`.

## Manual build

To build and run locally:
```sh
docker build --build-arg ALLOY_TAG=<tag> -t my-alloy:local .
docker run --rm -it my-alloy:local
```

## GitHub Actions

- The workflow is defined in `.github/workflows/rebuild-on-upstream.yml`.
- It uses:
  - A shell step to fetch the latest upstream tag with `curl` and `jq`.
  - [docker/login-action](https://github.com/docker/login-action) for GHCR authentication.
  - [docker/build-push-action](https://github.com/docker/build-push-action) to build and push the image.
  - [stefanzweifel/git-auto-commit-action](https://github.com/stefanzweifel/git-auto-commit-action) to update the last built tag.

## Customization

- Edit `alloy-config.yaml` to change the configuration.
- Edit the Dockerfile if you need to change the config path or add more files.
