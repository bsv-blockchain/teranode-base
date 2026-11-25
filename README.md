# teranode-base

This repo contains Dockerfiles for both 'build' as well as 'run' container images used as a base for the Teranode project, plus a 'runner' image for Teranode's self-hosted CI runners (see [Runner image](#runner-image)).  

These container images exist to install build and runtime dependencies for the application.  
They are sourced at the top of the main Teranode Dockerfile at [Dockerfile](https://github.com/bsv-blockchain/teranode/blob/main/Dockerfile).  

## Builds

Check new changes into the repo to get the CICD process to run. Please test the Dockerfiles locally before pushing changes to the repo.  

There are no automated tests for these builds at this time. [Todo: Add tests]

## Runner image

`Dockerfile_runner` builds the image that Teranode's self-hosted runners use. They run on Kubernetes through GitHub's Actions Runner Controller (ARC), which starts one runner pod per job from this image. It extends the stock [`ghcr.io/actions/actions-runner`](https://github.com/actions/runner/pkgs/container/actions-runner) image with tools Teranode's CI jobs expect from GitHub-hosted runners:

- `build-essential` (gcc, g++, `make`, libc and libstdc++ headers) for cgo, which `go test -race` and gobdk need, and for `make` targets
- the `docker compose` CLI plugin (Docker Compose 2.38.2, the version GitHub-hosted Ubuntu 24.04 runners ship), checked against its SHA-256. It needs a Docker daemon, which on ARC is the Docker-in-Docker sidecar of the runner pod
- `zstd`, so `actions/cache` compresses with zstd as on GitHub-hosted runners. `actions/cache` matches caches by compression method too, so without zstd jobs miss every cache saved with it
- `xz-utils`, `wget` and `bc`, small tools that scripts commonly assume, for example to fetch and unpack `.tar.xz` releases

Go and Node are not baked in: each job installs the version it needs with `actions/setup-go` and similar actions. The image still runs as the `runner` user (uid 1001).

It is built for amd64 only, by `.github/workflows/teranode_base_runner.yaml`, on pushes to `main` that touch `Dockerfile_runner` or that workflow, or by a manual run. Each build on `main` publishes:

- `ghcr.io/bsv-blockchain/teranode-base:runner-<short-sha>`: use this one in the runner configuration
- `ghcr.io/bsv-blockchain/teranode-base:runner-latest`
- the same two with an `-amd64` suffix, which the shared build workflow pushes first

`<short-sha>` is the abbreviated commit SHA of the build, as shown in the workflow run and in the package's tag list.

The runner image is rebuilt only when `Dockerfile_runner` or its workflow changes. The build and run images keep rebuilding on every push to `main`, except a push that touches only those two runner files.

### Bumping the actions-runner version

ARC runners don't update themselves, and GitHub expects self-hosted runners to move to a new runner release within 30 days of it coming out, so bump this regularly.

1. Pick the new release from [actions/runner releases](https://github.com/actions/runner/releases) and update the `FROM ghcr.io/actions/actions-runner:<version>` line in `Dockerfile_runner`.
2. Build and check it locally:
   ```
   docker build --platform linux/amd64 -f Dockerfile_runner -t teranode-runner .
   docker run --rm --platform linux/amd64 teranode-runner bash -c 'id; gcc --version; make --version; docker compose version; zstd --version; ls /home/runner/run.sh'
   ```
3. Merge to `main`, then change the image tag in the runner scale set configuration (kept outside this repo) to the new `runner-<short-sha>`. Rolling back is pointing it at the previous tag.

To bump Docker Compose, change `COMPOSE_VERSION` and `COMPOSE_SHA256` together. The SHA-256 is the digest of the `docker-compose-linux-x86_64` asset on the [release page](https://github.com/docker/compose/releases). Matching the version in [GitHub's Ubuntu 24.04 image](https://github.com/actions/runner-images/blob/main/images/ubuntu/Ubuntu2404-Readme.md) keeps jobs behaving the same on both kinds of runner.
