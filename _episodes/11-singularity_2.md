---
title: "Using Singularity"
teaching: 10
exercises: 5
questions:
- "How can I use CMSSW inside a container on LXPLUS?"
objectives:
- "Understand some of the differences between Singularity and Docker."
- "Successfully run a custom analysis container on LXPLUS."
keypoints:
- "Singularity needs to be used on LXPLUS."
- "CMS Computing provides a wrapper script to run CMSSW in different Linux environments (SLC5, SLC6, CC7, CC8)."
- "To run your own container, you need to run Singularity manually."
---

The previous episode has given you an idea how complicated it can be
to run containers with CVMFS access on your computer. However, at the
same time it gives you the possibility to develop code on a computer
that doesn't need to know anything about CMS software in the first place.
The only requirement is that Docker is installed.

You will also have noticed that in several cases *privileged* containers
are needed. These are not available to you on LXPLUS (nor is the `docker`
command). On LXPLUS, the tool to run containers is Singularity.
**The following commands will therefore all be run on LXPLUS**
(`lxplus7.cern.ch` or later specifically).

## CMS documentation on Singularity

Before we go into any detail, you should be aware of the
[central CMS documentation][cms-singularity]. These commands are only
available via `/cvmfs/cms.cern.ch/common`. The `cmssw-env` command is
actually a shell script that sets some variables automatically and then
runs Singularity. The nice thing about Singularity is that you can
mount `/cvmfs`, `/eos`, and `/afs` without any workarounds. This is
automatically done when running the `cmssw-env` command.

> ## Exercise: Run the CC7 Singularity container
>
> Confirm that you can access your EOS home directory
> (`/eos/user/${USER:0:1}/${USER}`) from the Singularity CC7 shell.
>
{: .challenge}

> ## Solution: Run the CC7 Singularity container
>
> ~~~
> cmssw-cc7
> ls /eos/user/${USER:0:1}/${USER}
> exit
> ~~~
> {: .language-bash}
>
{: .solution}

## Running custom images with Singularity

The CMS script discussed above is "nice-to-have" and works well
if you simply want to run some CMSSW code on a different Linux
distribution, but it also hides a lot of the complexity when
running Singularity. For the purpose of running your analysis
image, we cannot use the script above, but instead need to
run Singularity manually.

As an example, we are going to run the image that we used for
getting the VOMS proxy in the GitLab CI session. Before running
Singularity, mind that you should set the cache directory, i.e.
the directory to which the images are being pulled, to a
place outside your AFS space (here we use the `tmp` directory):

~~~
export SINGULARITY_CACHEDIR="/tmp/$(whoami)/singularity"
singularity shell -B /afs -B /eos -B /cvmfs docker://cmssw/cc7:latest
source /cvmfs/cms.cern.ch/cmsset_default.sh
~~~
{: .language-bash}

If you are asked for a docker username and password, just hit
enter twice. If you get an error message such as:

~~~
FATAL:   While making image from oci registry: failed to get checksum for docker://cmssw/cc7:latest: unable to retrieve auth token: invalid username/password
~~~
{: .output}

this is just a
[Singularity bug](https://github.com/sylabs/singularity/issues/4224).
To fix it, just delete the `~/.docker/config.json` file.

If you are past the authentication issue, you will get to see a
lot of garbage output and the `singularity shell` command will still
fail. The reason for this is a
[bug in Singularity](https://github.com/sylabs/singularity/issues/4943).

One particular difference to note w.r.t. to Docker is that the image
name needs to be prepended by `docker://` to tell Singularity that this
is a Docker image.
As you can see from the output, Singularity first downloads the layers
from the registry, and is then unpacking the layers into a format that
can be read by Singularity. This is somewhat a technical detail, but
this step is what fails at the moment (and is different w.r.t. Docker).

~~~
ERROR:   build: failed to make environment dirs: mkdir /tmp/clange/rootfs-ef013f60-51c7-11ea-bbe0-fa163e528257/.singularity.d: permission denied
FATAL:   While making image from oci registry: while building SIF from layers: packer failed to pack: while inserting base environment: build: failed to make environment dirs: mkdir /tmp/clange/rootfs-ef013f60-51c7-11ea-bbe0-fa163e528257/.singularity.d: permission denied
~~~
{: .output}

Once there is a new Singularity version (check via
`singularity --version`) more recent than
`3.5.2-1.1.el7` this will hopefully be fixed. For now, we cannot
use Singularity in this way. Otherwise, we'd be able to use
the `shell` to develop code interactively, and then use
`exec` to execute a script such as yesterday's `build.sh`
script:

~~~
export SINGULARITY_CACHEDIR="/tmp/$(whoami)/singularity"
singularity exec -B /afs -B /eos -B /cvmfs docker://cmssw/cc7:latest bash .gitlab/build.sh
~~~
{: .language-bash}

> ## `exec` vs. `shell`
>
> Singularity differentiates between providing you with an
> interactive shell (`singularity shell`) and executing scripts
> non-interactively (`singularity exec`).
>
{: .callout}

## Authentication with Singularity

In case your image is not public, you can authenticate to
the registry in two different ways: either you append the
option `--docker-login` to the `singularity` command, which
makes sense when running interactively, or via environment
variables (e.g. on GitLab):

~~~
export SINGULARITY_DOCKER_USERNAME=${CERNUSER}
export SINGULARITY_DOCKER_PASSWORD='mysecretpass'
~~~
{: .language-bash}

In the following episode we will try to work around the issues
observed above by using a very nice way to access unpacked images
directly via CVMFS.

{% include links.md %}

