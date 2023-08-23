---
title: "Using unpacked.cern.ch"
teaching: 10
exercises: 5
questions:
- "What is `unpacked.cern.ch`?"
- "How can I use `unpacked.cern.ch`?"
objectives:
- "Understand how your images can be put on `unpacked.cern.ch`"
keypoints:
- "The `unpacked.cern.ch` CVMFS area provides a very fast way of distributing unpacked docker images for access via Apptainer."
- "Using this approach you can run versioned and reusable stages of your analysis."
---
As was pointed out in the previous episode, Apptainer uses *unpacked* Docker
images. These are by default unpacked into the current working directory,
and the path can be changed by setting the `APPTAINER_CACHEDIR` variable.

The EP-SFT group provides a service that unpacks Docker images and makes them
available via a dedicated CVMFS area. In the following, you will learn how to
add your images to this area. Once you have your image(s) added to this area,
these images will be automatically synchronized from the image registry
to the CVMFS area within a few minutes whenever you create a new version of the image.

# Exploring the CVMFS `unpacked.cern.ch` area

The *unpacked* area is a directory structure within CVMFS:

~~~
ls /cvmfs/unpacked.cern.ch/
~~~
{: .language-bash}

~~~
gitlab-registry.cern.ch  registry.hub.docker.com
~~~
{: .output}

You can see the full directory structure of an image:

~~~
ls /cvmfs/unpacked.cern.ch/registry.hub.docker.com/fnallpc/fnallpc-docker:tensorflow-2.12.0-gpu-singularity/
~~~
{: .language-bash}

~~~
bin   dev          etc   lib    libgpuarray  mnt  proc  run   singularity  sys  usr
boot  environment  home  lib64  media        opt  root  sbin  srv          tmp  var
~~~
{: .output}

This can be useful for investigating some internal details of the image.

As mentioned above, the images are synchronized with the respective registry.
However, you don't get to know when the synchronization happened[^1], but there
is an easy way to check by looking at the time-stamp of the image directory:

~~~
ls -l /cvmfs/unpacked.cern.ch/registry.hub.docker.com/fnallpc/fnallpc-docker:tensorflow-2.12.0-gpu-singularity
~~~
{: .language-bash}

~~~
lrwxrwxrwx 1 cvmfs cvmfs 79 Apr 17 19:12 /cvmfs/unpacked.cern.ch/registry.hub.docker.com/fnallpc/fnallpc-docker:tensorflow-2.12.0-gpu-singularity -> ../../.flat/7b/7b4794b494eaee76f7c03906b4b6c1174da8589568ef31d3f881bdf820549161
~~~
{: .output}

In the example given here, the image has last been updated on August 17th at 13:54.

# Adding to the CVMFS `unpacked.cern.ch` area

You can add your image to the `unpacked.cern.ch` area by making a merge
request to the [unpacked sync repository][unpacked-sync]. In this repository
there is a file called [`recipe.yaml`][unpacked-sync-recipe], to which you
simply have to add a line with your full image name (including registry)
prepending `https://`:

~~~
    - 'https://registry.hub.docker.com/fnallpc/fnallpc-docker:tensorflow-2.12.0-gpu-singularity'
~~~
{: .language-yaml}

As of 14th February 2020, it is also possible to use wildcards for the
tags, i.e. you can simply add

~~~
    - 'https://registry.hub.docker.com/fnallpc/fnallpc-docker:*'
~~~
{: .language-yaml}

and whenever you build an image with a new tag it will be synchronized
to `/cvmfs/unpacked.cern.ch`.

> ## Image removal
> There is currently no automated ability to remove images from CVMFS. If you would like your image to be permanently removed, contact the developers or open a GitLab issue.
> 
{: .callout}

# Running Apptainer using the `unpacked.cern.ch` area

Running Apptainer using the `unpacked.cern.ch` area is done using the
same commands as listed in the previous episode with the only difference
that instead of providing a `docker://` image name to Apptainer,
you provide the path in `/cvmfs/unpacked.cern.ch`:

~~~
apptainer exec -B `readlink $HOME` -B `readlink -f ${HOME}/nobackup/` -B /cvmfs /cvmfs/unpacked.cern.ch/registry.hub.docker.com/fnallpc/fnallpc-docker:tensorflow-2.12.0-gpu-singularity /bin/bash
~~~
{: .language-bash}

Now you should be in an interactive shell almost immediately without any
image pulling or unpacking.

> ## Note
>
> Mind that you cannot change/write files into the container file system with Apptainer. If your activity will create or modify files in the container you will need to write those files to EOS or a mounted directory.
>
{: .callout}

> ## Where to go from here?
>
> Knowing that you can build images on your local machine, Docker Hub, GitHub, or GitLab and have them synchronized to the `unpacked.cern.ch` area, you now have the power to run reusable and versioned stages of your analysis. While we have only run these containers locally, you can [run them on the batch system][batchdocs-containers], i.e. your full analysis in containers with effectively only advantages.
>
{: .testimonial}

[^1]: You can figure it out by looking at the [Jenkins logs](https://lcgapp-services.cern.ch/cvmfs-jenkins/job/unpacked.cern.ch/).

{% include links.md %}

