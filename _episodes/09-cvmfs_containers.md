---
title: "Light-weight CMSSW containers"
teaching: 15
exercises: 10
questions:
- "How can I get a more light-weight CMSSW container?"
- "What are the caveats of using a light-weight CMSSW container?"
objectives:
- "Understand how the light-weight CMS containers can be used."
keypoints:
- "The light-weight CMS containers need to mount CVMFS."
- "They will only work with CVMFS available."
---
In a similar way to [running CMSSW in GitLab][gitlab-cms-lesson], the
images containing only the base operating system (e.g. Scientific Linux 5/6
or CentOS 7/8) plus additionally required system packages can be used to run
CMSSW (and other related software). CMSSW needs to be mounted via CVMFS.
**This is the recommended way!**

For this lesson, we will continue with the repository we used for the
[GitLab CI for CMS lesson][gitlab-cms-lesson] and just add to it.

## Adding analysis code to a light-weight container

Instead of using these containers only for compiling and
[running CMSSW][gitlab-cms-lesson-running], we can add our (compiled) code to
those images, building on top of them. The advantage in doing so is that you
will effectively be able to run your code in a version-controlled sandbox, in
a similar way as grid jobs are submitted and run. Adding your code on top of
the base image will only increase their size by a few Megabytes. CVMFS will
be mounted in the build step and also whenever the container is executed.
The important conceptual difference is that we do not use a *Dockerfile* to
build the image since that would not have CVMFS available, but instead we use
Docker *manually* as if it was installed on a local machine.

The way this is done is by requesting a **docker-privileged** GitLab runner.
With such a runner we can run Docker-in-Docker, which allows to manually
attach CVMFS to a container and run commands such as compiling analysis code
in this container. Compiling code will add an additional layer to the
container, which consists only of the effect of the commands run. After
exiting this container, we can tag this layer and push the container to the
container registry.

The *YAML* required looks as follows:

~~~
build_docker:
  only:
    - pushes
    - merge_requests
  tags:
    - docker-privileged
  image: docker:19.03.1
  services:
  # To obtain a Docker daemon, request a Docker-in-Docker service
  - docker:19.03.1-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_BUILD_TOKEN $CI_REGISTRY
    # Need to start the automounter for CVMFS:
    - docker run -d --name cvmfs --pid=host --user 0 --privileged --restart always -v /shared-mounts:/cvmfsmounts:rshared gitlab-registry.cern.ch/vcs/cvmfs-automounter:master
  script:
    # ls /cvmfs/cms.cern.ch/ won't work, but from the container it will
    # If you want to automount CVMFS on a new docker container add the volume config /shared-mounts/cvmfs:/cvmfs:rslave
    - docker run -v /shared-mounts/cvmfs:/cvmfs:rslave -v $(pwd):$(pwd) -w $(pwd) --name ${CI_PROJECT_NAME} ${FROM} /bin/bash ./.gitlab/build.sh
    - SHA256=$(docker commit ${CI_PROJECT_NAME})
    - docker tag ${SHA256} ${TO}
    - docker push ${TO}
  variables:
    FROM: gitlab-registry.cern.ch/clange/cmssw-docker/cc7-cms:latest
    TO: ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHORT_SHA}
~~~
{: .language-yaml}

This is pretty complicated, so let's break this into smaller pieces.

The `only` section determines when the step is actually run. The default
should probably be `pushes` only so that a new image is built whenever there
are changes to a branch. If you would like to build a container already when
a merge request is created so that you can test the code before merging, also
add `merge_requests` as in the example provided here.

The next couple of lines are related to the special Docker-in-Docker runner.
For this to work, the runner needs to be **privileged**, which is achieved by
adding `docker-privileged` to the `tags`. The image to run is then
`docker:19.03.1`, and in addition a special `service` with the name
`docker:19.03.1-dind` is required.

Once the runner is up, the `before_script` section is used to prepare the
setup for the following steps. First, the runner logs in to the GitLab image
registry with an automatically provided token (this is a property of the job
and does not need to be set by you manually). The second command starts a
special container, which mounts CVMFS and makes it available to our analysis
container.

In the `script` section the analysis container is then started, doing the following:

- mounting the volume (`-v /shared-mounts/cvmfs:/cvmfs:rslave`),
- mounting the current working directory (`-v $(pwd):$(pwd)`),
- setting the current working directory mounted as working directory inside the container (`-w $(pwd)`),
- settings its name to the project name (`--name ${CI_PROJECT_NAME}`),
- and executing the command `/bin/bash ./.gitlab/build.sh`.

The name of the image that is started is set via the `${FROM}` variable,
which is set to be `gitlab-registry.cern.ch/clange/cmssw-docker/cc7-cms:latest` here.

After the command that has been run in the container exits, a new *commit*
will have been added to the container. We can find out the hash of this
commit by running `docker commit ${CI_PROJECT_NAME}` (this is why we set the
container name to `${CI_PROJECT_NAME}`). With the following command, we then
*tag* this commit with the repository's registry name and a unique hash that
corresponds to the `git` commit at which we have built the image. This allows
for an easy correpondence between container name and source code version. The last command simply pushed this image to the registry.

> ## Exercise: Compile the `ZPeakAnalysis` inside the container
>
> The one thing that has not yet been explained is what the `build.sh` script
> does. This file needs to be part of the repository. Take the `ZPeakAnalysis`
> directory from yesterday's lesson, add it to the repository, and compile
> the code by adding the required commands to the `build.sh` script.
>
{: .challenge}

> ## Solution: Compile the `ZPeakAnalysis` inside the container
>
> A possible solution could look like this:
>
> ~~~
> #!/bin/bash
>
> # exit when any command fails; be verbose
> set -ex
>
> # make cmsrel etc. work
> shopt -s expand_aliases
> export MY_BUILD_DIR=${PWD}
> source /cvmfs/cms.cern.ch/cmsset_default.sh
> cd /home/cmsusr
> cmsrel CMSSW_10_6_8_patch1
> mkdir -p CMSSW_10_6_8_patch1/src/AnalysisCode
> mv ${MY_BUILD_DIR}/ZPeakAnalysis CMSSW_10_6_8_patch1/src/AnalysisCode
> cd CMSSW_10_6_8_patch1/src
> cmsenv
> scram b
> ~~~
> {: .language-bash}
>
{: .solution}

Since developing this using GitLab is tricky, the next episodes will cover
how you can develop this interactively on LXPLUS using Singularity or on
your own computer running Docker.
The caveat of using these light-weight images is that they cannot be run
autonomously, but always need to have CVMFS access to do anything useful.

> ## Why the `.gitlab` directory?
>
> Putting the `build.sh` script into a directory called `.gitlab` is a
> recommended convention. If you develop code locally (e.g. on LXPLUS), you
> will have a different directory structure. Your analysis code, i.e.
> `ZPeakAnalysis`, will reside within `CMSSW_10_6_8_patch1/src/AnalysisCode`, > and executing the script from within the `ZPeakAnalysis` does not make much
> sense, because you will create a CMSSW workarea within an existing one.
> Therefore, using a hidden directory with a name that makes it clear that
> this is for running within GitLab, and is ignored otherwise, can be useful.
>
{: .testimonial}

{% include links.md %}
