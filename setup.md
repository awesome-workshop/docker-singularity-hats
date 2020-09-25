---
title: Setup
---

## Docker installation

To install Docker Community Edition on your Linux, Mac, or Windows 10 (Pro, Enterprise, and Education) machine follow the [instructions in the Docker docs](https://docs.docker.com/get-docker/). If you are using Windows 10 Home you will need to follow [this Docker doc](https://docs.docker.com/docker-for-windows/install-windows-home/). Fair warning, the Windows 10 Home installation is more involved and requires Windows Subsystem for Linux 2 (WSL2), among other intracacies.

## Docker Hub

To signup for Docker Hub, follow the instructions [here](https://hub.docker.com/signup).

## Docker image pulls

Once you've got docker up and running, do the following docker image pulls in advance to save waiting during the tutorial:

~~~bash
docker pull python:3.7-slim
docker pull sl:7
docker pull aperloff/cms-cvmfs-docker:latest
docker pull cmscloud/cc7-cvmfs
docker pull gitlab-registry.cern.ch/cms-cloud/cmssw-docker/cmssw_10_2_21-slc7_amd64_gcc700:2020-09-22-ef834977
~~~

Most of these images are relatively small (a few 100 MB at most). However, the last image is almost 8 GB and may take some time to download.

## OSX specific setup

Follow the direction to install [XQuartz](https://www.xquartz.org/). Once installed, start the program and navigate to XQuartz->Preferences->Security. Make sure that both the "Authenticate connections" and "Allow connections from network clients" checkboxes are selected. If you change any settings, you will need to restart XQuartz.

## Windows specific setup

If you are not using cygwin and want to use X11, you will need to install an xwindow program. A popular options for Windows are xming and 
VcXsrv, though there are others available.

If you would like to use cygwin, you will need to install [winpty](https://github.com/rprichard/winpty) and prefix your docker command like `winpty docker`.

{% include links.md %}