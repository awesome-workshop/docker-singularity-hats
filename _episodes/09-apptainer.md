---
title: "Running Containers on CMSLPC/LXPLUS Using Apptainer"
teaching: 20
exercises: 10
questions:
- "How can I run a container on CMSLPC/LXPLUS?"
objectives:
- "Understand some of the differences between Apptainer and Docker."
- "Successfully run a custom analysis container on CMSLPC/LXPLUS."
- "Learn about the LPC Apptainer containers for machine learning."
keypoints:
- "Apptainer needs to be used for running containers on CMSLPC/LXPLUS."
- "CMS Computing provides a wrapper script to run CMSSW in different Linux environments (SLC5, SLC6, CC7, CC8)."
- "The centrally supported way to run CMSSW in a container is using Apptainer."
- "To run your own container, you need to run Apptainer manually."
---

# Introduction

<img align="right" src="../fig/Apptainer-logo.png" alt="Apptainer Logo" style="width:250px">

One thing that has not been covered in detail is that containers do not
necessarily have to be executed using Docker. While Docker is the most popular containerization tool these days, there are several so-called
container run-times that allow the execution of containers. The one chosen by CMS for invisibly running CRAB/HTCondor pilot jobs is [Apptainer][apptainer-docs], which is [centrally supported and documented][cms-apptainer]. The main reason for choosing Apptainer is that it is popular in high-performance and high-throughput computing and does not require any root privileges.

While executing images on CMSLPC/LXPLUS and HTCondor is more practical with
Apptainer, running in GitLab CI is by default done using Docker. Since
Apptainer uses a proprietary image format, but supports reading and
executing Docker images, building images is better done using Docker.

The previous episode has given you an idea how complicated it can be
to run containers with CVMFS access on your computer. However, at the
same time it gives you the possibility to develop code on a computer
that doesn't need to know anything about CMS software in the first place.
The only requirement is that Docker is installed. These same container can then be run in GitLab CI/CD via Docker.

You will also have noticed that in several cases *privileged* containers
are needed. These are not available to you on CMSLPC/LXPLUS (nor is the `docker` command). On CMSLPC/LXPLUS, the tool to run containers is Apptainer. **The following commands will therefore all be run on CMSLPC** (`cmslpc-sl7.fnal.gov` or specifically).

> ## Docker vs Apptainer
> Apptainer does not required a privileged daemon process, so it is usable by non-`root` users on shared clusters, which are commonly employed in HEP.
>
> <img align="center" src="../fig/DockerVsApptainer.png" alt="Docker vs. Apptainer" style="width:800px">
{: .callout}

> ## From Singularity to Apptainer
> Apptainer is free, community-supported container software that is part of the Linux Foundation.
> It was formerly known as Singularity, an open-source project started at Lawrence Berkeley National Laboratory.
> The name Singularity is still used by an alternate, commercial version of the project by the company Sylabs.
> More details about this change can be found [here](https://apptainer.org/news/community-announcement-20211130).
>
> The transition from Singularity to Apptainer is intended to be seamless.
> The commmand `singularity` is now aliased to `apptainer`,
> and `SINGULARITY` environment variables will be used if the corresponding `APPTAINER` environment variables are not defined.
> More details on backwards compatibility and relevant changes can be found [here](https://apptainer.org/docs/user/latest/singularity_compatibility.html).
{: .callout}

# CMS documentation on Apptainer

Before we go into any detail, you should be aware of the
[central CMS documentation][cms-apptainer]. These commands are only
available via `/cvmfs/cms.cern.ch/common`. The `cmssw-env` command is
actually a shell script that sets some variables automatically and then
runs Apptainer. The nice thing about Apptainer is that you can
mount `/cvmfs`, `/eos`, and `/afs` without any workarounds. This is
automatically done when running the `cmssw-env` command.

> ## Exercise: Run the CC8 Apptainer container
>
> Confirm that you can access your home directory (`~` or `${HOME}`), EOS area
> (`/eos/uscms/store/user/${USER}` on cmslpc), and 'nobackup' directory (`${HOME}/nobackup` on cmslpc) from within the Apptainer CC8 shell.
>
> > ## Solution: Run the CC8 Apptainer container
> >
> > ~~~bash
> > cat /etc/redhat-release
> > cmssw-cc8 --bind `readlink -f ${HOME}/nobackup/`
> > cat /etc/redhat-release
> > ls /eos/uscms/store/user/${USER}
> > ls ${HOME}
> > ls ${HOME}/nobackup
> > exit
> > ~~~
> > {: .source}
> {: .solution}
{: .challenge}

# Running custom images with Apptainer

The CMS script discussed above is "nice-to-have" and works well if you simply want to run some CMSSW code on a different Linux distribution, but it also hides a lot of the complexity when running Apptainer. For the purpose of running your analysis image, we cannot use the script above, but instead need to run Apptainer manually.

As an example, we are going to run a container using the `ubuntu:latest` image. Before running Apptainer, you should set the cache directory (i.e.
the directory to which the images are being pulled) to a
place outside your `$HOME`/AFS space (here we use the `~/nobackup` directory):

~~~bash
export APPTAINER_CACHEDIR="`readlink -f ~/nobackup/`/.apptainer/cache"
apptainer shell -B `readlink $HOME` -B `readlink -f ${HOME}/nobackup/` -B /cvmfs docker://ubuntu:latest
# try accessing cvmfs inside of the container
source /cvmfs/cms.cern.ch/cmsset_default.sh
~~~
{: .source}

~~~
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Getting image source signatures
Copying blob 345e3491a907 done
Copying blob 57671312ef6f done
Copying blob 5e9250ddb7d0 done
Copying config 7c6bc52068 done
Writing manifest to image destination
Storing signatures
2021/06/08 01:10:42  info unpack layer: sha256:345e3491a907bb7c6f1bdddcf4a94284b8b6ddd77eb7d93f09432b17b20f2bbe
2021/06/08 01:10:44  info unpack layer: sha256:57671312ef6fdbecf340e5fed0fb0863350cd806c92b1fdd7978adbd02afc5c3
2021/06/08 01:10:44  info unpack layer: sha256:5e9250ddb7d0fa6d13302c7c3e6a0aa40390e42424caed1e5289077ee4054709
INFO:    Creating SIF file...
INFO:    Converting SIF file to temporary sandbox...
WARNING: underlay of /etc/localtime required more than 50 (66) bind mounts
~~~
{: .output}

If you are asked for a docker username and password, just hit
enter twice.

It's not really a great practice to bind `/eos/uscms` into the container and you really shouldn't need to use the EOS fuse mount anyway.

One particular difference from Docker is that the image name needs to be prepended by `docker://` to tell Apptainer that this is a Docker image. Apptainer has its own registry system, which doesn't have a de facto default registry like [Docker Hub][docker-hub].

As you can see from the output, Apptainer first downloads the layers
from the registry, and is then unpacking the layers into a format that
can be read by Apptainer, the Singularity Image Format (SIF). This is a somewhat technical detail, but is different from Docker. It then unpacks the SIF file into  what it calls a *sandbox*, the uncompressed image files needed to make the container.

> ## `-B` (bind strings)
>
> The -B option allows the user to specify paths to bind to the Apptainer container. This option is similar to '-v' in docker. By default paths are mounted as rw (read/write), but can also be specified as ro (read-only).
>
> You must bind any mounted file systems to which you would like access (i.e. `nobackup`).
> 
> If you would like Apptainer to run your `.bashrc` file on startup, you must bind mount your home directory.
{: .callout}

In the next example, we are executing a script with Apptainer using the same image.

~~~bash
export APPTAINER_CACHEDIR="`readlink -f ~/nobackup/`/.apptainer/cache"
echo -e '#!/bin/bash\n\necho "Hello World!"\n' > hello_world.sh
apptainer exec -B `readlink $HOME` -B `readlink -f ${HOME}/nobackup/` docker://ubuntu:latest bash hello_world.sh
~~~
{: .source}

> ## `exec` vs. `shell`
>
> Apptainer differentiates between providing you with an
> interactive shell (`apptainer shell`) and executing scripts
> non-interactively (`apptainer exec`).
>
{: .callout}

## Saving the Apptainer Sandbox

You may have noticed that Apptainer caches both the Docker and SIF images so that they don't need to be pulled/created on subsequent Apptainer calls. That said, the sandbox needed to be created each time we started a container. If you will be using the same container multiple times, it may be useful to store the sandbox and use that to start the container.

Begin by building and storing the sandbox:

~~~bash
export APPTAINER_CACHEDIR="`readlink -f ~/nobackup/`/.apptainer/cache"
apptainer build --sandbox ubuntu/ docker://ubuntu:latest
~~~
{: .source}

~~~
INFO:    Starting build...
Getting image source signatures
Copying blob 345e3491a907 skipped: already exists
Copying blob 57671312ef6f skipped: already exists
Copying blob 5e9250ddb7d0 [--------------------------------------] 0.0b / 0.0b
Copying config 7c6bc52068 done
Writing manifest to image destination
Storing signatures
2021/06/08 01:13:24  info unpack layer: sha256:345e3491a907bb7c6f1bdddcf4a94284b8b6ddd77eb7d93f09432b17b20f2bbe
2021/06/08 01:13:24  warn xattr{etc/gshadow} ignoring ENOTSUP on setxattr "user.rootlesscontainers"
2021/06/08 01:13:24  warn xattr{/uscms_data/d2/aperloff/build-temp-220308461/rootfs/etc/gshadow} destination filesystem does not support xattrs, further warnings will be suppressed
2021/06/08 01:13:47  info unpack layer: sha256:57671312ef6fdbecf340e5fed0fb0863350cd806c92b1fdd7978adbd02afc5c3
2021/06/08 01:13:47  info unpack layer: sha256:5e9250ddb7d0fa6d13302c7c3e6a0aa40390e42424caed1e5289077ee4054709
INFO:    Creating sandbox directory...
INFO:    Build complete: ubuntu/
~~~
{: .output}

Once we have the sandbox we can use that when starting the container. Run the same command as before, but use the sandbox rather than the Docker image:

~~~bash
export APPTAINER_CACHEDIR="`readlink -f ~/nobackup/`/.apptainer/cache"
apptainer exec -B `readlink $HOME` -B `readlink -f ${HOME}/nobackup/` ubuntu/ bash hello_world.sh
~~~
{: .source}

~~~
WARNING: underlay of /etc/localtime required more than 50 (66) bind mounts
Hello World!
~~~
{: .output}

You will notice that the startup time for the container is significantly reduced.

## Authentication with Apptainer

In case your image is not public, you can authenticate to
the registry in two different ways: either you append the
option `--docker-login` to the `apptainer` command, which
makes sense when running interactively, or via environment
variables (e.g. on GitLab):

~~~
export APPTAINER_DOCKER_USERNAME=${CERNUSER}
export APPTAINER_DOCKER_PASSWORD='mysecretpass'
~~~
{: .language-bash}

Knowing how to authenticate will be important when pulling images from GitLab. For example:

~~~bash
export APPTAINER_CACHEDIR="`readlink -f ~/nobackup/`/.apptainer/cache"
apptainer shell -B `readlink $HOME` -B `readlink -f ${HOME}/nobackup/` docker://gitlab-registry.cern.ch/[repo owner's username]/[repo name]:[tag] --docker-login
~~~
{: .source}

# Custom containers for machine learning at the LPC

There are three GPUs available for use at the LPC. Rather than installing the myriad of packages necessary for machine learning on the bare-metal machines or manually synchronizing an Anaconda distribution to CVMFS (as done in the past), custom Apptainer images have been built for this purpose.

Currently we have several images built and ready to use:
1. [PyTorch][pytorch]: (latest [Dockerfile](https://github.com/FNALLPC/fnallpc-docker/blob/singularity-compatible/PyTorch/Dockerfile))
    1. 1.8.1 w/ CUDA 11.1
    1. 1.9.0 w/ CUDA 11.1
    1. 1.13.0 w/ CUDA 11.6
    1. 2.0.0 w/ CUDA 11.7
1. [TensorFlow][tensorflow]: (latest [Dockerfile](https://github.com/FNALLPC/fnallpc-docker/blob/singularity-compatible/TensorFlow/Dockerfile))
    1. 2.6.0 w/ GPU support
    1. 2.10.0 w/ GPU support
    1. 2.12.0 w/ GPU support

Of course there are many more packages installed in these images than just PyTorch or TensorFlow. If you don't see the exact variety you need, tell us and we can probably make it in ~1 day. The images are stored on [Docker Hub][docker-hub-fnallpc-docker].

> ## Access to the GPU
> 
> In order to give the container access to the CUDA drivers and GPU on the host machine you will need to add the `--nv` option to your `apptainer` command.
{: .callout}

> ## Using Jupyter within the image
> 
> You can start Jupyter Notebook using the `apptainer exec` command, passing the directive `jupyter notebook --no-browser --port <port_number>` at the end of the command. Apptainer, unlike Docker, doesn't require explicit port mapping. For example:
> ~~~bash
> apptainer exec --nv --bind $PWD:/run/user --bind `readlink -f ${HOME}/nobackup/` /cvmfs/unpacked.cern.ch/registry.hub.docker.com/fnallpc/fnallpc-docker:pytorch-1.8.1-cuda11.1-cudnn8-runtime-singularity jupyter notebook --no-browser --port 1234
> ~~~
> {: .source}
{: .callout}

A word of warning, these images are rather large (about 5 GB compressed). For that reason, it would be prohibitively expensive to make each user download, convert, and uncompress the images. Therefore, the unpacked sandboxes are stored on CVMFS at `/cvmfs/unpacked.cern.ch/registry.hub.docker.com/fnallpc/`. The service which does the unpacking and uploading to CVMFS will be discussed in detail in the next lesson. For now we'll just say, use the unpacked images if working with Apptainer.

> ## Exercise: PyTorch workflow
> Try to run a simple PyTorch workflow on a machine with a GPU
> > ## Solution
> > Log into one of the cmslpc GPU nodes and start a PyTorch container.
> > ~~~bash
> > ssh -Y <username>@cmslpcgpu<2-3>.fnal.gov
> > export APPTAINER_CACHEDIR="`readlink -f ~/nobackup/`/.apptainer/cache"
> > apptainer shell --nv --bind $PWD:/run/user --bind `readlink -f ${HOME}/nobackup/` /cvmfs/unpacked.cern.ch/registry.hub.docker.com/fnallpc/fnallpc-docker:pytorch-1.8.1-cuda11.1-cudnn8-runtime-singularity
> > ~~~
> > {: .source}
> > Create a script called `testPytorch.py` which has the following content:
> > ~~~python
> > #!/usr/bin/env python
> >
> > import torch
> > from datetime import datetime
> > 
> > print("torch.version.cuda = ",torch.version.cuda)
> > print("torch._C._cuda_getCompiledVersion() = ",torch._C._cuda_getCompiledVersion())
> >
> > for i in range(10):
> >     x = torch.randn(10, 10, 10, 10) # similar timings regardless of the tensor size
> >     t1 = datetime.now()
> >     x.cuda()
> >     print(i, datetime.now() - t1)
> > ~~~
> > {: .source}
> > Execute that script using:
> > ~~~bash
> > python testPytorch.py
> > ~~~
> > {: .source}
> > You should see an output which looks similar to:
> > ~~~
> > torch.version.cuda =  11.1
> > torch._C._cuda_getCompiledVersion() =  11010
> > 0 0:00:45.501856
> > 1 0:00:00.000083
> > 2 0:00:00.000066
> > 3 0:00:00.000041
> > 4 0:00:00.000041
> > 5 0:00:00.000040
> > 6 0:00:00.000038
> > 7 0:00:00.000039
> > 8 0:00:00.000041
> > 9 0:00:00.000039
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

> ## Other useful Apptainer images for CMS analyzers
>  There are some additional, ready-made Apptainer images which might be useful to know about. Those include:
>  - [Pre-built coffea + scientific python environment](https://coffeateam.github.io/coffea/installation.html#pre-built-images)
>    - Great if you want a ready to use environment with all of the most popular dependencies pre-installed
>    - There are also Docker images available
>  - [SONIC for Triton Inference Server](https://github.com/cms-sw/cmssw/blob/master/HeterogeneousCore/SonicTriton/README.md)
>    - The image runs the Nvidia Triton inference server needed by SONIC
>    - Can access local CPU and/or GPU resources
>    - Can be run using Docker instead of Apptainer if needed
> 
{: .testimonial}

> ## Going deeper
> 
> Follow the [bonus lesson on advanced usage of Apptainer]({{ page.root }}{% link _episodes/19-apptainer-advanced.md %}) to learn more!
{: .callout}

[tensorflow]: https://www.tensorflow.org/
[pytorch]: https://pytorch.org/
[docker-hub-fnallpc-docker]: https://hub.docker.com/repository/docker/fnallpc/fnallpc-docker/tags?page=1

{% include links.md %}
