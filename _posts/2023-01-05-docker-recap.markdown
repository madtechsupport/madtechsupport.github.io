---
layout: post
title:  "Docker recap"
date:   2023-01-05 17:00:00 +0000
---

I last seriously used Docker for the [WWF Endangered Emoji](https://slate.com/human-interest/2015/05/wwf-endangered-emoji-saving-endangered-animals-using-emoji-tweets.html) campaign and the [Happy Studio](https://www.campaignlive.com/article/mcdonalds-happy-studio-r-ga-london/1370520) global rollout both were 2015 projects and alot's changed since then.

I've done some reading and here's a short summary of the story as I see it:
* Containers become standards based. The introduction of the Open Container Initiative [OCI](https://opencontainers.org/) and [runC](https://www.docker.com/blog/runc/) in 2015.
* The Linux kernel gets [cgroups v2](https://en.wikipedia.org/wiki/Cgroups) and [Containerd](https://www.docker.com/blog/introducing-containerd/) arrives 2016
* The [Moby project](https://www.docker.com/blog/introducing-the-moby-project/) and [Docker CE](https://docker-docs.netlify.app/release-notes/docker-ce/#17030-ce-2017-03-01) are launched in 2017 by which time there is already a large and growing number of container related project.
* Red Hat made changes to [Fedora 31](https://www.redhat.com/sysadmin/fedora-31-control-group-v2) in 2019 that kind of [broke things for Docker on Fedora](https://fedoramagazine.org/docker-and-fedora-32/) but improved things for [Podman](https://www.redhat.com/en/topics/containers/what-is-podman).

Fast forward to 2023 where Docker would like you to do all your development using [Docker Desktop](https://www.docker.com/products/docker-desktop/alternatives/) because that's there business model and other vendors like RedHat support a container ecosystem that's built around cgroups v2 and OCI Containers using Podman as the drop-in replacement for Docker.

# Installing Docker on Fedora
The Fedora 37 Workstation I'm using doesn't come with `docker` installed, so if you want to use `docker` it needs to be installed. The alternative is to use `podman`, but for now I'm installing `docker`. The options for installing `docker` on Fedora are:
* [Docker Desktop for Linux](https://docs.docker.com/desktop/install/linux-install/)
* [Docker Engine](https://docs.docker.com/engine/install/fedora/)
* [Docker Community Edition](https://developer.fedoraproject.org/tools/docker/docker-installation.html)

I don't want to install Docker Desktop because it's heavy weight, commercial and much more than I need. On closer examination, Docker Engine and Docker Community Edition are the _same thing_ just with different names (the instructions for installing Docker Engine are basically the same as for installing Docker CE with the addition of installing the `docker-compose-plugin`).

So I followed the instructions for installing Docker Engine/Docker CE for Fedora and started running `docker` commands. However, each time I want to run `docker` I need to also use `sudo` and that starts to get annoying. I think I'll try out `podman` one of these days to see how that goes.
