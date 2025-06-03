# teranode-base

This repo contains Dockerfiles for both 'build' as well as 'run' container images used as a base for the Teranode project.  

These container images exist to install build and runtime dependencies for the application.  
They are sourced at the top of the main Teranode Dockerfile at [Dockerfile](https://github.com/bitcoin-sv/teranode/blob/main/Dockerfile).  

## Builds

Check new changes into the repo to get the CICD process to run. Please test the Dockerfiles locally before pushing changes to the repo.  

There are no automated tests for these builds at this time. [Todo: Add]
