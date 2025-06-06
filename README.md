<div align="center">

## Containers

_An opinionated collection of container images_

</div>

<div align="center">

![GitHub Repo stars](https://img.shields.io/github/stars/first-it-consulting/containers?style=for-the-badge)
![GitHub forks](https://img.shields.io/github/forks/first-it-consulting/containers?style=for-the-badge)
![GitHub Workflow Status (with event)](https://img.shields.io/github/actions/workflow/status/first-it-consulting/containers/release.yaml?style=for-the-badge&label=Release)

</div>

Welcome to our container images, if looking for a container start by [browsing the GitHub Packages page for this repo's packages](https://github.com/orgs/first-it-consulting/packages?repo_name=containers).

This project is heavily inspired from [home-operations](https://github.com/home-operations/containers/tree/main).

This is almost a fork of the the home-operations project with the aim to have my own images available for my home lab cluster. However, I want also learn how things are working and got inspired from this project.

## Mission statement

The goal of this project is to support [semantically versioned](https://semver.org/), [rootless](https://rootlesscontaine.rs/), and [multiple architecture](https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/) containers for various applications.

It also adheres to a [KISS principle](https://en.wikipedia.org/wiki/KISS_principle), logging to stdout, [one process per container](https://testdriven.io/tips/59de3279-4a2d-4556-9cd0-b444249ed31e/), no [s6-overlay](https://github.com/just-containers/s6-overlay) and all images are built on top of [Alpine](https://hub.docker.com/_/alpine) or [Ubuntu](https://hub.docker.com/_/ubuntu).

## Features

### Tag immutability

The containers built here do not use immutable tags, as least not in the more common way you have seen from [linuxserver.io](https://fleet.linuxserver.io/) or [Bitnami](https://bitnami.com/stacks/containers).

We do take a similar approach but instead of appending a `-ls69` or `-r420` prefix to the tag we instead insist on pinning to the sha256 digest of the image, while this is not as pretty it is just as functional in making the images immutable.

| Container                                                              | Immutable |
|------------------------------------------------------------------------|-----------|
| `ghcr.io/first-it-consulting/gitlab-runner:rolling`                    | ❌         |
| `ghcr.io/first-it-consulting/gitlab-runner:4.0.13.2932`                | ❌         |
| `ghcr.io/first-it-consulting/gitlab-runner:rolling@sha256:8053...`     | ✅         |
| `ghcr.io/first-it-consulting/gitlab-runner:4.0.13.2932@sha256:8053...` | ✅         |

_If pinning an image to the sha256 digest, tools like [Renovate](https://github.com/renovatebot/renovate) support updating the container on a digest or application version change._

### Rootless

To run these containers as non-root make sure you update your configuration to the user and group you want.

#### Docker compose

```yaml
networks:
  gitlab-runner:
    name: gitlab-runner
    external: true
services:
  gitlab-runner:
    image: ghcr.io/first-it-consulting/gitlab-runner:4.0.13.2932
    container_name: gitlab-runner
    user: 65534:65534
    # ...
```

#### Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab-runner
# ...
spec:
  # ...
  template:
    # ...
    spec:
      # ...
      securityContext:
        runAsUser: 65534
        runAsGroup: 65534
        fsGroup: 65534
        fsGroupChangePolicy: OnRootMismatch
# ...
```

### Passing arguments to a application

Some applications do not support defining configuration via environment variables and instead only allow certain config to be set in the command line arguments for the app. To circumvent this, for applications that have an `entrypoint.sh` read below.

1. First read the Kubernetes docs on [defining command and arguments for a Container](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/).
2. Look up the documentation for the application and find a argument you would like to set.
3. Set the extra arguments in the `args` section like below.

    ```yaml
    args:
      - --port
      - "8080"
    ```

### Configuration volume

For applications that need to have persistent configuration data the config volume is hardcoded to `/config` inside the container. This is not able to be changed in most cases.

### Verify image signature

These container images are signed using the [attest-build-provenance](https://github.com/actions/attest-build-provenance) action.

The attestations can be checked with the following command, verifying that the image is actually built by the GitHub CI system:

```sh
gh attestation verify --repo first-it-consulting/containers oci://ghcr.io/first-it-consulting/${APP}:${TAG}
```

### Eschewed features

There is no multiple "channels" of the same application. For example Prowlarr, Radarr, Lidarr, and Sonarr will only have the develop branch published and not the master "stable" branch. Qbittorrent will only be published with LibTorrent 2.x.

## Contributing

We encourage the use of upstream container images whenever possible. However, there are scenarios where using an upstream image may not be feasible—such as when tools like [s6-overlay](https://github.com/just-containers/s6-overlay) or [gosu](https://github.com/tianon/gosu) are needed, or when the application exhibits [unusual behaviors](https://github.com/nzbgetcom/nzbget/blob/989e848f8e9d3d4031f5d09d7b8945954a9f67b0/docker/entrypoint.sh#L17-L18), particularly when running as [root](https://github.com/plexinc/pms-docker/blob/8a42ea4c623e4df06928f945bcf8f450ba77fcf5/root/etc/cont-init.d/45-plex-hw-transcode-and-connected-tuner#L21). In such cases, contributing to this repository might make sense.

If you’re considering contributing, please note that **we only accept pull requests** under the following conditions:

- The upstream application is actively maintained,
- **and** there is no official upstream container,
- **or** the official image does not support multi-architecture builds,
- **or** the official image relies on tools like s6-overlay, gosu, or other unconventional initialization mechanisms.

## Deprecations

Containers here can be **deprecated** at any point, this could be for any reason described below.

1. The upstream application is **no longer actively developed**
2. The upstream application has an **official upstream container** that follows closely to the mission statement described here
3. The upstream application has been **replaced with a better alternative**
4. The **maintenance burden** of keeping the container here **is too bothersome**

**Note**: Deprecated containers will remained published to this repo for 6 months after which they will be pruned.

## Credits

A lot of inspiration and ideas are thanks to the hard work of the home-ops community, [hotio.dev](https://hotio.dev/) and [linuxserver.io](https://www.linuxserver.io/) contributors.
