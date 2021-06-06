---
title: "Pulling Images"
teaching: 10
exercises: 5
questions:
- "How are images downloaded?"
- "How are images distinguished?"
objectives:
- "Pull images from Docker Hub image registry"
- "List local images"
- "Introduce image tags"
keypoints:
- "Pull images with `docker pull`"
- "List images with `docker images`"
- "Image tags distinguish releases or version and are appended to the image name with a colon"
- "The default registry is Docker Hub"
---

<figure align="right" style="display: table; margin: 0; text-align: center;">
<iframe scrolling="no"  src="https://videos.cern.ch/video/OPEN-VIDEO-2021-119-003" width="560" height="315" frameborder="0" allowfullscreen></iframe>
<br/>
<figcaption style="display: table-caption; caption-side: bottom; background: pink; padding: 10px;">Recording of the HATS@LPC2020 session (<a href="https://videos.cern.ch/video/OPEN-VIDEO-2021-119-003">link</a>). Note, you must have a CERN login to watch this video.</figcaption>
</figure>

# Docker Hub and Image Registries

Much like GitHub allows for web hosting and searching for code, the [Docker Hub][docker-hub] image registry allows the same for Docker images. Hosting and building of images is [free for public repositories][docker-hub-billing] and allows for downloading images as they are needed. Additionally, through integrations with GitHub and Bitbucket, Docker Hub repositories can be linked against Git repositories so that [automated builds of Dockerfiles on Docker Hub][docker-hub-builds] will be triggered by pushes to repositories.

<img align="center" src="../fig/High-level-overview-of-Docker-architecture.png" alt="High-level overview of the Docker architecture" style="width:600px">

While Docker Hub is maintained by Docker and is the defacto default registry for Docker images, it is not the only registry in existence. There are many registries, both private and public, in existence. For example, the GitLab software allows for a registry service to be setup alongside its Git and CI/CD management software. CERN's GitLab instance has such a registry available. See later episodes for more information on CERN's GitLab Docker image registry.

# Pulling Images

To begin with we're going to [pull][docker-docs-pull] down the Docker image we're going to be working in for the tutorial (**Note:** If you did all the docker pulls in the setup instructions, this image will already be on your machine. In this case, docker should notice it's there and not attempt to re-pull it, unless the image has changed in the meantime.):

~~~bash
docker pull sl

#if you run into a premission error, use "sudo docker run ..." as a quick fix
# to fix this for the future, see https://docs.docker.com/install/linux/linux-postinstall/
~~~
{: .source}

~~~
Using default tag: latest
latest: Pulling from library/sl
32cc6c378ee5: Pull complete
Digest: sha256:6cc5f47d16bd74ea8a2c3f68ead5ee893c032ccd75269d6d714cb34fc2153cb7
Status: Downloaded newer image for sl:latest
docker.io/library/sl:latest
~~~
{: .output}

The image names are composed of `NAME[:TAG|@DIGEST]`, where the `NAME` is composed of `REGISTRY-URL/NAMESPACE/IMAGE` and is often referred to as a *repository*. Here are some things to know about specifying the image:
* Some repositories will include a `USERNAME` as part of the image name (i.e. `fnallpc/fnallpc-docker`), and others, usually Docker verified content, will include only a single name (i.e. `sl`).
* A registry path (`REGISTRY-URL/NAMESPACE`) is similar to a URL, but does not contain a protocol specifier (https://). Docker uses the https:// protocol to communicate with a registry, unless the registry is allowed to be accessed over an insecure connection. Registry credentials are managed by docker login. If no registry path is given, the docker daemon assumes you meant to pull from Docker Hub and automatically appends `docker.io/library` to the beginning of the image name.
* If no tag is provided, Docker Engine uses the `:latest` tag as a default.
* The SHA256 `DIGEST` is much like a Git hash, where it allows you to pull a specific version of an image. 
* CERN GitLab's repository path is `gitlab-registry.cern.ch/<username>/<repository>/<image_name>[:<tag>|@<digest>]`.

Now, let's [list the images][docker-docs-images] that we have available to us locally

~~~bash
docker images
~~~
{: .source}

If you have many images and want to get information on a particular one you can apply a filter, such as the repository name

~~~bash
docker images sl
~~~
{: .source}

~~~
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
sl                  latest              dddf32954161        3 weeks ago         177MB
~~~
{: .output}

or more explicitly

~~~bash
docker images --filter=reference="sl"
~~~
{: .source}

~~~
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
sl                  latest              dddf32954161        3 weeks ago         177MB
~~~
{: .output}

You can see here that there is the `TAG` field associated with the
`sl` image.
Tags are way of further specifying different versions of the same image.
As an example, let's pull the `7` release tag of the
[sl image](https://hub.docker.com/_/sl) (again, if it was already pulled during setup, docker won't attempt to re-pull it unless it's changed since last pulled).

~~~bash
docker pull sl:7
docker images sl
~~~
{: .source}

~~~
7: Pulling from library/sl
Digest: sha256:6cc5f47d16bd74ea8a2c3f68ead5ee893c032ccd75269d6d714cb34fc2153cb7
Status: Downloaded newer image for sl:7
docker.io/library/sl:7

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
sl                  7                   dddf32954161        3 weeks ago         177MB
sl                  latest              dddf32954161        3 weeks ago         177MB
~~~
{: .output}

> ## Pulling Python
>
> Pull the image python:3.7-slim for Python 3.7 and then list all `python` images along with the `sl:7` image
>
> > ## Solution
> >
> > ~~~bash
> > docker pull python:3.7-slim
> > docker images --filter=reference="sl" --filter=reference="python"
> > ~~~
> > {: .source}
> >
> > ~~~
> > 3.7-slim: Pulling from library/python
d121f8d1c412: Pull complete
ca572574cc82: Pull complete
2bec6349c99d: Pull complete
087ac0b72728: Pull complete
6ca52d7c92b3: Pull complete
Digest: sha256:e787b48ee93cad4d7157e13c01c109650ddad8f622fab6644ab5dd700eacae64
Status: Downloaded newer image for python:3.7-slim
docker.io/library/python:3.7-slim
> > 
> > REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
> > python              3.7-slim            4d4a9832278b        2 weeks ago         112MB
> > sl                  7                   dddf32954161        3 weeks ago         177MB
> > sl                  latest              dddf32954161        3 weeks ago         177MB
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

[docker-hub]: https://hub.docker.com/
[docker-hub-billing]: https://hub.docker.com/billing-plans/
[docker-hub-builds]: https://docs.docker.com/docker-hub/builds/
[docker-docs-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-docs-images]: https://docs.docker.com/engine/reference/commandline/images/

{% include links.md %}
