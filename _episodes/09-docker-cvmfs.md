---
title: "Accessing CVMFS From Docker Locally"
teaching: 10
exercises: 0
questions:
- "How can I access CVMFS from my computer?"
- "How can I access CVMFS from Docker?"
objectives:
- "Be aware of CVMFS with Docker options."
- "Successfully mount CVMFS via a privileged container."
keypoints:
- "It is more practical to use light-weight containers and obtain CMSSW via CVMFS."
- "You can install CVMFS on your local computer."
- "The `cvmfs-automounter` allows you to provide CVMFS to other containers on Linux."
- "Privileged containers can be dangerous."
- "You can mount CVMFS from within a container on container startup."
---
An alternative to full CMSSW release images are Linux images that only contain the underlying base operating system (e.g. Scientific Linux 5/6/7 or CentOS 7/8), including additionally required system packages. These images have a size of a few hundred Megabytes, but rely on a good network connection to access the CVMFS share.

In order to use CVMFS via Docker, a couple of extra steps need to be taken.
There are different approaches:

1. Installing CVMFS on your computer locally and mounting it from the container.
1. Mounting CVMFS via another container and providing it to the analysis container.
1. Mounting CVMFS from within the analysis container.

We will go through these options in the following.

> ## This is where things get ugly
>
> Unfortunately, all the options have some caveats, and they might not
> even work on your computer. At the moment, no clear recommendations can be
> given. Try for yourself which option works best for you.
{: .callout}

# Installing CVMFS on the host

CVMFS can be installed locally on your computer. The installation
packages are provided on the [CVMFS Downloads page][cvmfs-download]. In the
interest of time, we will not install CVMFS now, but instead use the third
option above in the following exercises. If you would like to install CVMFS on your computer, make sure to read the [CVMFS Client Quick Start Guide][cvmfs-quickstart]. Please also have a look at the [CVMFS with Docker documentation][cvmfs-docker-docs] to avoid common pitfalls when running Linux on your computer and trying to bind mount CVMFS from the host. This is not necessary when running on a Mac. However, on a Mac you need to go to *Docker Settings* -> *Resources* -> *File Sharing* and add `/cvmfs` to enable bind mounting.

![`/cvmfs` needs to be accessible to Docker for bind mounting](../fig/docker_file_sharing.png)

To run your analysis container and give it access to the `/cvmfs` mount, run
the following command (remember that `--rm` deletes the container after exiting):

~~~
docker run --rm -it -v /cvmfs:/cvmfs gitlab-registry.cern.ch/cms-cloud/cmssw-docker/cc7-cms /bin/bash
~~~
{: .language-bash}

> ## Limitations
> I'm told that mounting cvmfs locally on Windows only works in WSL2 and that the Linux instructions have to be used.
{: .callout}

# Using the `cvmfs-automounter`

The first option needed CVMFS to be installed on the host computer (i.e. your laptop, a GitLab runner, or a Kubernetes node). Using the `cvmfs-automounter` is effectively mimicking what is done on the CERN GitLab CI/CD runners. First, a container, the `cvmfs-automounter`, is started that mounts CVMFS, and then this container provides the CVMFS mount to other containers. This is very similar to how modern web applications are orchestrated. If you are running Linux, the following command should work.
**On a OSX and Windows+cygwin, however, this will not work** (at least at the moment). This could work if you are using [Windows Subsystem for Linux 2 (WSL2)][wsl2] in combination with [Docker for WSL2][wsl2-docker], but not cygwin.

~~~
sudo mkdir /shared-mounts
docker run -d --name cvmfs --pid=host --user 0 --privileged --restart always -v /shared-mounts:/cvmfsmounts:rshared gitlab-registry.cern.ch/vcs/cvmfs-automounter:master
~~~
{: .language-bash}

This container is running as a daemon (`-d`), but you can still see it via
`docker ps` and also kill it using `docker kill cvmfs`.

To mount CVMFS inside your analysis container use the following command:

~~~
docker run -v /shared-mounts/cvmfs:/cvmfs:rslave -v $(pwd):$(pwd) -w $(pwd) --name ${CI_PROJECT_NAME} ${FROM} /bin/bash
~~~
{: .language-bash}

> ## The downside to mounting CVMFS inside a container
>
> 1. The CVMFS cache will remain as long as the container is around. If the container is removed, so will the cache. This means it could take longer for commands to run the first time they are called after mounting CVMFS. This same caveat holds true for the methods which are discussed below.
{: .callout}

# Mounting CVMFS inside the analysis container

This method seems to work on OSX, Windows 10 Pro, and most Linux systems. For the most part, it does not rely on the host system configuration. The caveat is that the container runs with elevated privileges, but if you trust me, you can use it.

Like the full CMSSW images, the centrally produced, light-weight images are created by a [build service (in development)][cms-containers] and are hosted at [CERN GitLab][cms-cloud-gitlab] and currently mirrored at [Docker Hub][cms-cloud-docker-hub].

We can start by running one of these light weight images.

~~~
docker run --rm -it --cap-add SYS_ADMIN --device /dev/fuse gitlab-registry.cern.ch/cms-cloud/cmssw-docker/cc7-cvmfs bash
~~~
{: .language-bash}

If you get an error similar to:

~~~
/bin/sh: error while loading shared libraries: libtinfo.so.5: failed to map segment from shared object: Permission denied
~~~
{: .output}

you need to turn off SElinux security policy enforcing:

~~~
sudo setenforce 0
~~~
{: .language-bash}

This can be changed permanently by editing `/etc/selinux/config`, setting `SELINUX` to `permissive` or `disabled`. Mind, however, that there are certain security issues with disabling SElinux security policies as well as running privileged containers.

> ## The downsides to starting CVMFS in the container
>
> 1. The CVMFS daemon is started when the container is started for the first
> time. It is not started again when you e.g. lose your network connection
> or simply connect back to the container at a later stage. At that point,
> you won't have CVMFS access anymore.
> 1. Every container will have their own instance of the CVMFS daemon and the associated cache. It would be more efficient to share a single mount and cache.
> 1. The automounting capabilities are up to the image creator and not provided by the CVMFS development team.
{: .callout}

> ## Exercise: Give it a try!
> Try if you can run the following command from your cloned repository base
> directory:
> ~~~
docker run --rm -it --cap-add SYS_ADMIN --device /dev/fuse gitlab-registry.cern.ch/cms-cloud/cmssw-docker/cc7-cvmfs:latest bash
cmsrel CMSSW_10_2_21
cd CMSSW_10_2_21/src/
cmsenv
> ~~~
> {: .language-bash}
{: .challenge}

Currently there is something wrong with these centrally produced images when being run on OSX. You will most likely receive an error message, like the one below, when trying to start the container.
~~~
chgrp: invalid group: 'fuse'
::: cvmfs-config...
Failed to get D-Bus connection: Operation not permitted
Failed to get D-Bus connection: Operation not permitted
::: mounting FUSE...
Moint point /cvmfs/cms.cern.ch does not exist
Moint point /cvmfs/cms-opendata-conddb.cern.ch does not exist
::: mounting FUSE... [done]
::: Mounting CVMFS... [FAILED]
::: Did you run the docker container in privileged mode?
::: Setting up CMS environment...
/opt/cms/entrypoint.sh: line 7: /cvmfs/cms.cern.ch/cmsset_default.sh: No such file or directory
~~~
{: .output}

Nevertheless, these apparently work in Windows. They still print the error messages above, but the container is still able to mount CVMFS. This image hasn't been tested on Linux recently.

Current downsides to these images:
1. If the mounting of CVMFS fails the image immediately exits. You must change the entrypoint in order to debug the issue.
1. If the CVMFS daemon is interrupted there is no automatic way to reconnect.
1. The container has sudo privileges. In other words, the user permissions can be elevated. Not necessarily a bad thing, but something to be aware of.
1. There is little to no support for X11 or VNC.

> ## Developing CMS code on your laptop
>
> By using containers, you can effectively develop any and all HEP-related code
> (and beyond) on your local development machine, and it doesn't need to know
> anything about CVMFS or CMSSW in the first place.
>
{: .testimonial}

{% include links.md %}

[wsl2]: https://docs.microsoft.com/en-us/windows/wsl/wsl2-install
[wsl2-docker]: https://docs.docker.com/docker-for-windows/wsl-tech-preview/
