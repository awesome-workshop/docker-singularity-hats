---
title: "Containerizing a CMSSW Working Area"
teaching: 20
exercises: 20
questions:
- "How do I create an image which contains my CMSSW working area?"
- "How can I mount CVMFS inside that image?"
objectives:
- "Learn how to use the `containerize.sh` script to automate the process of creating OCI images from CMSSW directories."
keypoints:
- "Use the `containerize.sh` script within the `FNALLPC/lpc-scripts` package in order to make an image containing an arbitrary CMSSW directory."
---

Wouldn't it be great if you could create a container around your CMSSW work area and ship that off to a worker node or CI/CD system for use? Now you can! The [FNALLPC/lpc-scripts](https://github.com/FNALLPC/lpc-scripts/) repository contains a script for containerizing **any** CMSSW release. The script is called [`containerize.sh`](https://github.com/FNALLPC/lpc-scripts/blob/master/containerize/containerize.sh) and with just a few short commands it can have your CMSSW directory turned into an image in no time.

# Initial Demo Setup

The LPC has specific nodes dedicated to building software packages and containers. You can access these nodes by logging into one of the following machines:

* cmslpc-sl7-heavy.fnal.gov
* cmslpc-c8-heavy01.fnal.gov

While this process isn't strictly limited to the LPC computers, this is the environment where it was originally designed and tested.

For this demonstration we'll want to setup a CMSSW release on the CMSLPC computers. It doesn't matter which version or which packages it includes. Here we'll use `CMSSW_10_6_21_patch1` and checkout the `JetMETCorrections/Modules` package. Don't forget to compile the code.

~~~bash
ssh -Y <username>@cmslpc-sl7-heavy.fnal.gov
# cd to some working location
cmsrel CMSSW_10_6_28_patch1
cd CMSSW_10_6_28_patch1/src
cmsenv
git-cms-init
git-cms-addpkg JetMETCorrections/Modules
scram b -j 8
cd CMSSW_10_6_28_patch1/src/
cmsenv
~~~
{: .source}

Once that's setup, we can start to containerize the release.

> ## Useful Aliases
> You may wish to alias the buildah and podman commands as follows so that you don't have to set the `root` and `runroot` paths each time.
> 
> ~~~bash
> alias buildah='buildah --root /scratch/containers/`whoami`/ --runroot /scratch/containers/`whoami`/'
> alias podman='podman --root /scratch/containers/`whoami`/ --runroot /scratch/containers/`whoami`/'
> ~~~
> {: .source}
> 
> For the purposes of this demonstration we will also alias the `containerize.sh` shell script as follows:
> 
> ~~~bash
> alias containerize='/cvmfs/cms-lpc.opensciencegrid.org/FNALLPC/lpc-scripts/containerize/containerize.sh'
> ~~~
> {: .source}
{: .callout}

To see a full list of options for `containerize.sh` use the command:

~~~bash
containerize -h
~~~
{: .source}

# Containerize CMSSW and Mount External CVMFS Directory

To create the image from the CMSSW release, run a command similar to:

~~~bash
containerize -t containerize:demo1 -b docker://docker.io/aperloff/cms-cvmfs-docker:light -C
~~~
{: .source}

At this point we'll see that the image does indeed exist. Then we'll run the image and mount the host machines copy of CVMFS inside the container.

~~~bash
podman images -a
podman run --rm -it -v /cvmfs/cms.cern.ch/:/cvmfs/cms.cern.ch/:ro containerize:demo1
ls -alh ./
ls -alh /cvmfs/cms.cern.ch
~~~
{: .source}

The `ls` commands should show you the user area within the container as well as list the contents of the `/cvmfs/cms.cern.ch/` mount.

# Containerize CMSSW With Internal CVMFS Mount

In this case we will rely on the CVMFS mounting capabilities of the `aperloff/cms-cvmfs-docker` image. Create the image using the following command:

~~~bash
containerize -t containerize:demo2 -b docker://docker.io/aperloff/cms-cvmfs-docker:light
~~~
{: .source}

You can run a container with the resulting image, while mounting CVMFS, by using a command similar to:

~~~bash
podman run --rm -it -P --device /dev/fuse --cap-add SYS_ADMIN -e CVMFS_MOUNTS="cms.cern.ch" containerize:demo2
ls -alh ./
ls -alh /cvmfs/cms.cern.ch
~~~
{: .source}

You should similarly see the home area for the containers user and the `/cvmfs/cms.cern.ch/` mount.

# Save Resulting Images

If you'd like to save the resulting images for later use, you can either save the image to a tarball or push it to a registry. To save the image to a tarball use the command:

~~~bash
podman image save --compress -o "<name>.tar" localhost/<name>:<tag>
~~~
{: .source}

To save the image to a registry, use the commands:

~~~bash
podman login -u <DockerHub username> -p <DockerHub password> registry-1.docker.io
podman push localhost/<name>:<tag> <username>/<name>:<tag>
~~~
{: .source}

# Further Customizing the Builds

Although we expect that the default options for `containerize.sh` should be good enough for most users, there are still plenty of command line options which can be used to modify the default behavior. Additionally, if a user had the need to modify the image building commands, they might consider creating their own version of `/cvmfs/cms-lpc.opensciencegrid.org/FNALLPC/lpc-scripts/containerize/Dockerfile` and specify to use that file using the `-f` option. If additional directories need to be cached, then the user could make a copy of `/cvmfs/cms-lpc.opensciencegrid.org/FNALLPC/lpc-scripts/containerize/cache.json` and specify its location using the `-j` option.

{% include links.md %}