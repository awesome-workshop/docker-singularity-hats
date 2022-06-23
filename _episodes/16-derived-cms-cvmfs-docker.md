---
title: "Building derived images from the cms-cvmfs-docker base image"
teaching: 20
exercises: 20
questions:
- "Why is it harder to build derived images for this container?"
- "When do I need to use a workaround to build a derived image and when is it okay to use standard methods?"
- "Are there any limitations on what typed of derived images I can and cannot build?"
objectives:
- "Be able to build a derived image using the cms-cvmfs-docker image as a base and installing software taken from CVMFS."
- "Know when and how to use Moby BuildKit."
keypoints:
- "Moby BuildKit is a new build engine for Docker and includes a lot of new and advanced features."
- "You can mount CVMFS when building images, with some caveats."
---

It is possible to build derived images using the `cms-cmvfs-docker` image as a base. That said, there are some things you should know before you begin.

First an foremost, the CVMFS mount is not typically available during the build stage. The non-technical way of explaining this is that durring the build there is no running container and thus the mount points (i.e. CVMFS endpoints) aren't connected; you're just building an image layer by layer. This means that you can't easily access or setup any of the software on CVMFS during the build. However, there are two workarounds available.

The first method relies on the ability of docker to save the state of a running container as an image. The proceedure, in a nutshell, looks like:

1. Spin up a container using `aperloff/cms-cvmfs-docker:latest`.
2. wget, or download some other way, a setup script which specified all of the commands you would like to perform during the build. Typically these are the actions you would perform from within a Dockerfile.
3. Run the setup script to install/setup the environment/software.
4. (optional) clear the CVMFS cache.
5. Use `docker commit` to save the state of the container.
6. (optional) Push the new image to a container registry.

A set of example commands would look like:

~~~bash
# Run the container and perform the setup actions
docker run -t -P --device /dev/fuse --cap-add SYS_ADMIN -e CVMFS_MOUNTS="cms.cern.ch" --name myimage --entrypoint "/bin/bash" aperloff/cms-cvmfs-docker:latest -c "/run.sh -c \"wget <url to setup script>/setup.sh && chmod +x setup.sh && ./setup.sh\" && cvmfs_config wipecache"
# Create an image from the state of the container
docker commit -c 'ENTRYPOINT ["/run.sh"]' -c 'CMD []' myimage aperloff/myimage:latest
# Login to a container registry (i.e. DockerHub) if you intend to push an image
echo "<DockerHub password>" | docker login -u <DockerHub username> --password-stdin
# Push the image to a container registry (i.e. DockerHub)
docker push aperloff/myimage:latest
~~~
{: .source}

The second method is funcationally simpler, but relies on very recent features introduced for Docker. You will want to make sure that you have an updated Docker build (>=18.09). Recent versions of Docker come bundled with Moby BuildKit ([GitHub](https://github.com/moby/buildkit#used-by), [blog post](https://blog.mobyproject.org/introducing-buildkit-17e056cc5317)), which comes with a number of build enhancements.

There are several ways in which to specify that you would like to use the new build feature. If you would like to manually tell docker to use BuildKit, then you can append `DOCKER_BUILDKIT=1` in front of each build command (e.g. `DOCKER_BUILDKIT=1 docker build .`). If you would like to turn on BuildKit by default, then you can add `{ "features": { "buildkit": true } }` in the daemon configuration, either through Docker Desktop or by editing `/etc/docker/daemon.json` (default path on a Linux host). A third way is to directly call the BuildKit builder using the modified build command `docker buildx build`.

Once you have up-to-date build capabilities, you can create a Dockerfile which uses the latest frontend syntax features by adding a comment to the first line of the file:

~~~
# syntax=docker/dockerfile:1
~~~

This will make sure you're using the latest release of the version 1 syntax (i.e. the current version). The Dockerfile used to build the modified image would look something like:

~~~
# syntax=docker/dockerfile:1

FROM aperloff/cms-cvmfs-docker:latest

USER root

ARG ARG_CVMFS_MOUNTS
ARG ARG_MY_UID
ARG ARG_MY_GID

ENV CVMFS_MOUNTS=$ARG_CVMFS_MOUNTS
ENV MY_UID=$ARG_MY_UID
ENV MY_GID=$ARG_MY_GID

RUN --security=insecure source /mount_cvmfs.sh  && \
    mount_cvmfs && \
    ls /cvmfs/cms.cern.ch && \
    source /home/cmsusr/.bashrc && \
    <do domething interesting here> && \
    ls -alh

ENTRYPOINT ["/run.sh"]
~~~
There are a few pieces that deserve to be highlighted. First, notice that the USER specified is the `root` user. This is necessary so that the user inside the container has enough permission to mount CVMFS. This can potentially be lowered later using something like `su cmsusr`. However, note that you cannot split apart the portions of the `RUN` command as CVMFS is only mounted on that specific layer/shell. Additionally, notice that the `RUN` command is followed by `--security=insecure`, which allows for the mounting of CVMFS. Also note that anny configuration arguments you usually use to configure the CVMFS mount can be passed as build arguments. Finally, it doesn’t matter that CMSSW was checked out as the `root` user since `/run.sh` chowns all of the files in `/home/cmsusr`.

Once you have the Dockerfile for the derived image you can run the following commands to get an image named `cms-cvmfs-docker:derived`:

~~~bash
docker buildx create --driver-opt image=moby/buildkit:master --use --name insecure-builder --buildkitd-flags '--allow-insecure-entitlement security.insecure'
docker buildx use insecure-builder
docker buildx build --load --allow security.insecure --build-arg ARG_CVMFS_MOUNTS="cms.cern.ch oasis.opensciencegrid.org" --build-arg ARG_MY_UID=$(id -u) --build-arg ARG_MY_GID=$(id -g) -t cms-cvmfs-docker:derived .
docker buildx rm insecure-builder
~~~

Notice that similar build arguments are passed to the build command as you would use to start a container using the base image. The other important pieces are `--load` to save the output image into the local database. You could use `--push` to send the image directly to a registry. Then there is `--allow security.insecure`, which is needed to allow for the mounting of CVMFS.

Once the build is done using either of the methods mentioned above you can start a container using the same commands as before.

Please note, much of the information here was obtained from the following pages in the Docker docs:
* [Build images with BuildKit](https://docs.docker.com/develop/develop-images/build_enhancements/)
* [Docker Buildx](https://docs.docker.com/buildx/working-with-buildx/)
* [docker buildx build](https://docs.docker.com/engine/reference/commandline/buildx_build/)
If you have further questions about BuildKit syntax you might start by referencing:
* [BuildKit Syntax Reference](https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/syntax.md)
* [StackOverflow on Privileged buils with Docker and BuildKit](https://stackoverflow.com/questions/48098671/build-with-docker-and-privileged/72342814#72342814)

{% include links.md %}