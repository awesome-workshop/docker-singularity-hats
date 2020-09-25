---
title: "Using full CMSSW containers"
teaching: 10
exercises: 0
questions:
- "How can I obtain a standalone CMSSW container?"
objectives:
- "Understanding how to find and use standalone CMSSW containers."
keypoints:
- "Standalone CMSSW containers are currently not routinely built due to their size."
- "They need to be built/requested when needed."
---
As discussed in the
[introduction]({{ page.root }}{% link _episodes/01-introduction.md %}),
the images that contain full CMSSW releases can be very big. CMS computing
therefore does not routinely build these images. However, as part of the
[CMS Open Data effort][cms-opendata], images are provided for some releases.
You can find those on [Docker Hub][docker-cmsopendata]. In addition, a
[build service][cms-containers] is currently under development.

If you would like to use these images, you can use them in the same way as
the other CMS images with the only difference that the CMSSW software in the
container is in `/opt/cms` and not within `/cvmfs/cms.cern.ch`.

You can run the containers as follows (pick either `bash` or `zsh`) when
using the version published on Docker Hub:

~~~
docker run --rm -it cmsopendata/cmssw:10_6_8_patch1 /bin/zsh
~~~
{: .language-bash}

The images are in several cases also mirrored on the CERN GitLab registry:

~~~
docker run --rm -it gitlab-registry.cern.ch/clange/cmssw-docker/cmssw_10_6_8_patch1 /bin/zsh
~~~
{: .language-bash}

> ## Do not use for large-scale job submission nor on GitLab!
>
> Due to the large size of these images, they should only be used for local
> development.
>
{: .callout}

{% include links.md %}
