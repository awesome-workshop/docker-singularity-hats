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
- "The `unpacked.cern.ch` CVMFS area provides a very fast way of distributing unpacked docker images for access via Singularity."
- "Using this approach you can run versioned and reusable stages of your analysis."
---
As was pointed out in the previous episode, Singularity uses *unpacked* Docker
images. These are by default unpacked into the current working directory,
and the path can be changed by setting the `SINGULARITY_CACHEDIR` variable.

The EP-SFT group provides a service that unpacks Docker images and makes them
available via a dedicated CVMFS area. In the following, you will learn how to
add your images to this area. Once you have your image(s) added to this area,
these images will be automatically synchronised from the image registry
to the CVMFS area within a few minutes whenever
you create a new version of the image.

We will continue with the `ZPeakAnalysis` example, but for demonstration
purposes we will use an [example payload][payload-docker-cms].

## Exploring the CVMFS `unpacked.cern.ch` area

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
ls /cvmfs/unpacked.cern.ch/gitlab-registry.cern.ch/awesome-workshop/payload-docker-cms:3daaa96e
~~~
{: .language-bash}

~~~
afs  builds  dev          eos  home  lib64       media  opt   proc  run   singularity  sys  usr
bin  cvmfs   environment  etc  lib   lost+found  mnt    pool  root  sbin  srv          tmp  var
~~~
{: .output}

This can be useful for investigating some internal details of the image.

As mentioned above, the images are synchronised with the respective registry.
However, you don't get to know when the synchronisation happened, but there
is an easy way to check by looking at the timestamp of the image directory:

~~~
ls -l /cvmfs/unpacked.cern.ch/gitlab-registry.cern.ch/awesome-workshop/payload-docker-cms:3daaa96e
~~~
{: .language-bash}

~~~
lrwxrwxrwx. 1 cvmfs cvmfs 79 Feb 18 00:31 /cvmfs/unpacked.cern.ch/gitlab-registry.cern.ch/awesome-workshop/payload-docker-cms:3daaa96e -> ../../.flat/28/28ba0646b6e62ab84759ad65c98cab835066c06e5616e48acf18f880f2c50f90
~~~
{: .output}

In the example given here, the image has last been updated on February 18th at 00:31.

## Adding to the CVMFS `unpacked.cern.ch` area

You can add your image to the `unpacked.cern.ch` area by making a merge
request to the [unpacked sync repository][unpacked-sync]. In this repository
there is a file called [`recipe.yaml`][unpacked-sync-recipe], to which you
simply have to add a line with your full image name (including registry)
prepending `https://`:

~~~
- https://gitlab-registry.cern.ch/awesome-workshop/payload-docker-cms:3daaa96e
~~~
{: .language-yaml}

As of 14th February 2020, it is also possible to use wildcards for the
tags, i.e. you can simply add

~~~
- https://gitlab-registry.cern.ch/awesome-workshop/payload-docker-cms:*
~~~
{: .language-yaml}

and whenever you build an image with a new tag it will be synchronised
to `/cvmfs/unpacked.cern.ch`.

## Running Singularity using the `unpacked.cern.ch` area

Running Singularity using the `unpacked.cern.ch` area is done using the
same commands as listed in the previous episode with the only difference
that instead of providing a `docker://` image name to Singularity,
you provide the path in `/cvmfs/unpacked.cern.ch`:

~~~
singularity shell -B /afs -B /eos -B /cvmfs /cvmfs/unpacked.cern.ch/gitlab-registry.cern.ch/awesome-workshop/payload-docker-cms:3daaa96e
~~~
{: .language-bash}

Now you should be in an interactive shell almost immediately without any
image pulling or unpacking. One important thing to note is that for most
CMS images the default username is `cmsusr`, and if you compiled your
analysis code in the container, it will by default reside in
`/home/cmsusr`:

~~~
Singularity> cd /home/cmsusr/CMSSW_10_6_8_patch1/src/
Singularity> source /cvmfs/cms.cern.ch/cmsset_default.sh
Singularity> cmsenv
Singularity> cd AnalysisCode/ZPeakAnalysis/
Singularity> cmsRun test/MyZPeak_cfg.py
~~~
{: .language-bash}

And there we are, we run the analysis in a container interactively!
However, there is one issue we will run in. After running over the
input file, the `cmsRun` command will exit with a warning:

~~~
Warning in <TStorageFactoryFile::Write>: file myZPeak.root not opened in write mode
~~~
{: .output}

The output file will actually not be written. The reason for that is that
we cannot write into the container file system with Singularity.
We will have to change the `MyZPeak_cfg.py` file such that it writes
out to a different path.

> ## Challenge: Patch `MyZPeak_cfg.py` to write out to your EOS home
>
> Or even better, use an environment variable to define this that if not set
> defaults to `./`. Mind that you cannot change files in the container,
> so the way to go is change the python config in the repository and
> have a new image built that can then be used.
>
{: .challenge}

> ## Solution: Patch `MyZPeak_cfg.py` to write out to your EOS home
>
> A possible solution could look like this
> ~~~
> import os
> outPath = os.getenv("ANALYSIS_OUTDIR") + "/"
> if not outPath:
>   outPath = "./"
> process.TFileService = cms.Service("TFileService",
>                                    fileName = cms.string(outPath + 'myZPeak.root')
>                                    )
> ~~~
> {: .language-python}
{: .solution}

Commit these changes, push them, and your new image will show up on
CVMFS within a few minutes. The new image has the tag `0950e980`.

~~~
singularity shell -B /afs -B /eos -B /cvmfs /cvmfs/unpacked.cern.ch/gitlab-registry.cern.ch/awesome-workshop/payload-docker-cms:0950e980
Singularity> cd /home/cmsusr/CMSSW_10_6_8_patch1/src/
Singularity> source /cvmfs/cms.cern.ch/cmsset_default.sh
Singularity> cmsenv
Singularity> cd AnalysisCode/ZPeakAnalysis/
Singularity> export ANALYSIS_OUTDIR="/eos/user/${USER:0:1}/${USER}"
Singularity> cmsRun test/MyZPeak_cfg.py
Singularity> exit
ls -l /eos/user/${USER:0:1}/${USER}/myZPeak.root
~~~
{: .language-bash}

> ## Where to go from here?
>
> Knowing that you can build images on GitLab and have them synchronised to
> the `unpacked.cern.ch` area, you now have the power to run reusable and
> versioned stages of your analysis. While we have only run test jobs using
> these containers, you can
> [run them on the batch system][batchdocs-containers], i.e. your full analysis
> in containers with effectively only advantages.
The next step after this is to connect these stages
> using workflows, which will be taught tomorrow.
>
{: .testimonial}

{% include links.md %}

