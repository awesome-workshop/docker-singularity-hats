---
title: "Using Full CMSSW Containers"
teaching: 10
exercises: 0
questions:
- "How can I obtain a standalone CMSSW container?"
- "What are the advantages and disadvantages of this type of container?"
objectives:
- "Understanding how to find and use standalone CMSSW containers."
keypoints:
- "Full CMSSW release containers are very big."
- "Standalone CMSSW containers are currently not routinely built due to their size."
- "They need to be built/requested when needed."
---
CMS does not have a concept of separating analysis software from the rest of
the experimental software stack such as event generation, data taking, and
reconstruction. This means that there is just one *CMSSW*, and the releases
have a size of several Gigabytes (around 20 GB for the last releases).

From the user's and computing point of view, this makes it very impractical to
build and use images that contain a full CMSSW release. Imagine running
several hundred batch jobs where each batch node first needs to download
several Gigabytes of data before the job can start, amounting to a total of
tens of Terabytes. These images, however, can be useful for offline code
development, in case CVMFS is not available, as well as for overall
preservation of the software.

Because images that contain full CMSSW releases can be very big, CMS computing
does not routinely build these images. However, as part of the
[CMS Open Data effort][cms-opendata], images are provided for some releases.
You can find those on [Docker Hub][docker-cmsopendata]. In addition, a
[build service][cms-containers] is currently under development. Those images can be found on [CERN GitLab][cms-cloud-gitlab] and can be mirrored to [Docker Hub][docker-hub] upon request.

If you would like to use these images, you can use them in the same way as
any other CMS images (see next episode) with the only difference that the CMSSW software in the
container is in `/opt/cms` and not within `/cvmfs/cms.cern.ch`. The ability to partially replicate the `/cvmfs/cms.cern.ch` directory within these containers is under development. The goal being to have the same paths to CMSSW regardless of the way in which CMSSW is accessed within the container.

You can run the containers as follows (pick either `bash` or `zsh`) when
using the version published on [Docker Hub][docker-cmsopendata]:

~~~
docker run --rm -it cmsopendata/cmssw:10_6_8_patch1 /bin/zsh
~~~
{: .language-bash}

If you would like to use the images from the [CERN GitLab registry][cms-cloud-gitlab]:

~~~
docker run --rm -it gitlab-registry.cern.ch/cms-cloud/cmssw-docker/cmssw_10_2_21-slc7_amd64_gcc700 /bin/zsh
~~~
{: .language-bash}

> ## Do not use for large-scale job submission nor on GitLab!
>
> Due to the large size of these images, they should only be used for local
> development.
>
{: .callout}

> ## Note
> One important thing to note is that for most CMS images, the default username is `cmsusr`. This will hold true for all of the centrally produced CMS images mentioned in these lessons.
{: .callout}

{% include links.md %}
