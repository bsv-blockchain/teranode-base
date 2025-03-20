# teranode-base

This repo contains Dockerfiles for both 'build' as well as 'run' container images used as a base for the Teranode project.

These container images exist to install build and runtime dependencies for the application, and are sourced at the top of the main Teranode Dockerfile at https://github.com/bitcoin-sv/teranode/blob/main/Dockerfile

## Builds

CICD workflows are setup for manual dispatch at this time. Changes checked into this repo won't automatically trigger builds (yet). Go to the Actions tab to manually trigger a build for either the base build or run images.
