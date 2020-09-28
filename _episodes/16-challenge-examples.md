---
title: "Bonus: SSH Credentials"
teaching: 0
exercises: 10
questions:
- "How do I access my SSH credentials within a container?"
objectives:
- "How to run with SSH and not use `cp`"
keypoints:
- "Containers are very extensible"
---

> ## Get SSH credentials in a container without `cp`
>
> Get SSH credentials inside a container without using `cp`
>
> > ## Solution
> >
> > Mount multiple volumes
> >~~~
> >docker run --rm -it --device /dev/fuse --cap-add SYS_ADMIN \
> >  -e CVMFS_MOUNTS="none" \
> >  -e MY_UID=$(id -u) -e MY_GID=$(id -g) \
> >  -w /home/cmsusr/workdir \
> >  -v $PWD:/home/cmsusr/workdir \
> >  -v $HOME/.ssh:/home/cmsusr/.ssh \
> >  -v $HOME/.gitconfig:/home/cmsusr/.gitconfig \
> >  aperloff/cms-cvmfs-docker:latest
> >~~~
> >{: .source}
> {: .solution}
{: .challenge}

{% include links.md %}
