# Drone docker images for Rancher prime

This is a monorepo where you can find all the images used on the drone pipeline for `Rancher prime`.

Some images are used on the pipeline itself ( github-release, slack, docker, git...) and other are used
on the components of drone ( cleaner and runner)

## Why
In order to run `drone` pipelines for `Rancher prime`, we need to add some features to the base image.
In this monorepo, you can find the different components and their `Dockerfile`

## Triggering builds
To deliver new version of the components, is defined a `.drone.yml` which will be in charge of:
- Testing if the build of the images are working ( It's performing only a build of the image)
- Publishing the new image to the `rancher/drone-prime` docker repository.

The publish step will be executed when a `tag` is created on `main` branch.

### Tag naming
Due to the characteristic of `drone` and our infra settings. We decided to follow the the next tag namming:
**v+${version_number}+-+${component_name}**

The **${component_name}** will determine which folder will be selected to perform `docker build`

Examples:
- v0.0.1-manifest
- v1.2.5-publish

### Docker images
The docker images are published under https://hub.docker.com/r/rancher/drone-prime/tags.
The tag will be the same that the one which triggered the pipeline plus the sufix **-s390x**

Examples:

|   Git tag         |   Docker image                                |
|:------------------|:----------------------------------------------|
|   v0.0.1-git      |   rancher/drone-images:v0.0.1-git-s390x       |
|   v1.2.5-slack    |   rancher/drone-images:v1.2.5-slack-s390x     |
|   v3.20.1-docker  |   rancher/drone-images:v3.20.1-docker-s390x   |

 