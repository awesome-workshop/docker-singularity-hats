---
title: "Running Containers"
teaching: 10
exercises: 5
questions:
- "How are containers run?"
- "How do you monitor containers?"
- "How are containers exited?"
- "How are containers restarted?"
objectives:
- "Run containers"
- "Understand container state"
- "Stop and restart containers"
keypoints:
- "Run containers with `docker run`"
- "Monitor containers with `docker ps`"
- "Exit interactive sessions just as you would a shell, with `exit`"
- "Stop a container with `docker stop`"
- "Restart stopped containers with `docker start`"
---

To use a Docker image as a particular instance on a host machine you [run][docker-docs-run] it as a container. You can run in either a [detached or foreground][docker-docs-run-detached] (interactive) mode.

Run the image we pulled as a container with an interactive bash terminal:

~~~bash
docker run -it sl:7 /bin/bash
~~~
{: .source}

The `-i` option here enables the interactive session, the `-t` option gives access to a terminal and the `/bin/bash` command makes the container start up in a bash session. 

You are now inside the container in an interactive bash session. Check the file directory

~~~bash
pwd
ls -alh
~~~
{: .source}

~~~
/
total 56K
drwxr-xr-x   1 root root 4.0K Sep 25 08:26 .
drwxr-xr-x   1 root root 4.0K Sep 25 08:26 ..
-rwxr-xr-x   1 root root    0 Sep 25 08:26 .dockerenv
lrwxrwxrwx   1 root root    7 Sep  1 13:06 bin -> usr/bin
dr-xr-xr-x   2 root root 4.0K Apr 12  2018 boot
drwxr-xr-x   5 root root  360 Sep 25 08:26 dev
drwxr-xr-x   1 root root 4.0K Sep 25 08:26 etc
drwxr-xr-x   2 root root 4.0K Sep  1 13:07 home
lrwxrwxrwx   1 root root    7 Sep  1 13:06 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Sep  1 13:06 lib64 -> usr/lib64
drwxr-xr-x   2 root root 4.0K Apr 12  2018 media
drwxr-xr-x   2 root root 4.0K Apr 12  2018 mnt
drwxr-xr-x   2 root root 4.0K Apr 12  2018 opt
dr-xr-xr-x 210 root root    0 Sep 25 08:26 proc
dr-xr-x---   2 root root 4.0K Sep  1 13:07 root
drwxr-xr-x  11 root root 4.0K Sep  1 13:07 run
lrwxrwxrwx   1 root root    8 Sep  1 13:06 sbin -> usr/sbin
drwxr-xr-x   2 root root 4.0K Apr 12  2018 srv
dr-xr-xr-x  12 root root    0 Sep 23 04:12 sys
drwxrwxrwt   2 root root 4.0K Sep  1 13:07 tmp
drwxr-xr-x  13 root root 4.0K Sep  1 13:06 usr
drwxr-xr-x  18 root root 4.0K Sep  1 13:07 var
~~~
{: .output}

and check the host to see that you are not in your local host system

~~~bash
hostname
~~~
{: .source}

~~~
<generated hostname>
~~~
{: .output}

Further, check the `os-release` to see that you are actually inside a release of Scientific Linux:

~~~bash
cat /etc/os-release
~~~
{: .source}

~~~
NAME="Scientific Linux"
VERSION="7.8 (Nitrogen)"
ID="scientific"
ID_LIKE="rhel centos fedora"
VERSION_ID="7.8"
PRETTY_NAME="Scientific Linux 7.8 (Nitrogen)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:scientificlinux:scientificlinux:7.8:GA"
HOME_URL="http://www.scientificlinux.org//"
BUG_REPORT_URL="mailto:scientific-linux-devel@listserv.fnal.gov"

REDHAT_BUGZILLA_PRODUCT="Scientific Linux 7"
REDHAT_BUGZILLA_PRODUCT_VERSION=7.8
REDHAT_SUPPORT_PRODUCT="Scientific Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="7.8"
~~~
{: .output}

# Monitoring Containers

Open up a new terminal tab on the host machine and
[list the containers that are currently running][docker-docs-ps]

~~~bash
docker ps
~~~
{: .source}

~~~
CONTAINER ID        IMAGE         COMMAND             CREATED             STATUS              PORTS               NAMES
<generated id>      <image:tag>   "/bin/bash"         n minutes ago       Up n minutes                            <generated name>
~~~
{: .output}

> ## `container` command
> 
> You can also list the containers by using
> 
> ~~~bash
> docker container ls
> ~~~
>{: .source}
{: .callout}

Notice that the name of your container is some randomly generated name.
To make the name more helpful, [rename][docker-docs-rename] the running container

~~~bash
docker rename <CONTAINER ID> my-example
~~~
{: .source}

and then verify it has been renamed

~~~bash
docker ps
~~~
{: .source}

~~~
CONTAINER ID        IMAGE         COMMAND             CREATED             STATUS              PORTS               NAMES
<generated id>      <image:tag>   "/bin/bash"         n minutes ago       Up n minutes                            my-example
~~~
{: .output}

> ## Renaming by name
>
> You can also identify containers to rename by their current name
>
> ~~~bash
>docker rename <NAME> my-example
> ~~~
>{: .source}
{: .callout}

> ## Specifying a name
> 
> You can also startup a container with a specific name
> 
> ~~~bash
> docker run -it --name my-example sl:7 /bin/bash
> ~~~
>{: .source}
{: .callout}

# Exiting a container

As a test, go back into the terminal used for your container, and create a file in the container

~~~bash
touch test.txt
~~~
{: .source}

In the container exit at the command line

~~~bash
exit
~~~
{: .source}

You are returned to your shell.
If you list the containers you will notice that none are running

~~~bash
docker ps
~~~
{: .source}

~~~
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
~~~
{: .output}

but you can see all containers that have been run and not removed with

~~~bash
docker ps -a
~~~
{: .source}

~~~
CONTAINER ID        IMAGE         COMMAND             CREATED            STATUS                     PORTS               NAMES
<generated id>      <image:tag>   "/bin/bash"         n minutes ago      Exited (0) t seconds ago                       my-example
~~~
{: .output}

# Restarting a container

To restart your exited Docker container [start][docker-docs-start] it again and then [attach][docker-docs-attach] it interactively to your shell

~~~bash
docker start <CONTAINER ID>
docker attach <CONTAINER ID>
~~~
{: .source}

> ## Attach shortcut
> The two commands above (`docker start` and `docker attach`) can be combined into a single command as shown below:
> 
> ~~~bash
> docker start -ai <CONTAINER ID>
> ~~~
{: .callout}

> ## `exec` command
> The [attach][docker-docs-attach] command used here is a handy shortcut to interactively access a running container with the same start command (in this case `/bin/bash`) that it was originally run with. 
>
> In case you'd like some more flexibility, the [exec][docker-docs-exec] command lets you run any command in the container, with options similar to the run command to enable an interactive (`-i`) session, etc. 
>
> For example, the `exec` equivalent to `attach`ing in our case would look like:
> ~~~bash
> docker start <CONTAINER ID>
> docker exec -it <CONTAINER ID> /bin/bash
> ~~~
> 
> You can start multiple shells inside the same container using `exec`.
{: .callout}

> ## Starting and attaching by name
>
>You can also start and attach containers by their name
>
>~~~bash
>docker start <NAME>
>docker attach <NAME>
>~~~
>{: .source}
{: .callout}


Notice that your entry point is still `/` and then check that your
`test.txt` still exists

~~~bash
ls -alh test.txt
~~~
{: .source}

~~~
-rw-r--r-- 1 root root 0 Sep 25 08:39 test.txt
~~~
{: .output}

So this shows us that we can exit Docker containers for arbitrary lengths of time and then
return to our working environment inside of them as desired.

>## Clean up a container
>
>If you want a container to be [cleaned up][docker-docs-run-clean-up] &mdash; that is deleted &mdash; after you exit it then run with the `--rm` option flag
>
>~~~bash
>docker run --rm -it <IMAGE> /bin/bash
>~~~
>{: .source}
{: .callout}

# Stopping a container

Sometimes you will exited a container and it won't stop. Other times your container may crash or enter a bad state, but still be running. In order to stop a container you will exit it (`exit`) and then enter:

~~~bash
docker stop <CONTAINER ID> # or <NAME>
~~~
{: .source}

Stopping the container is a prerequisite for it's [removal]({{ page.root }}{% link _episodes/04-removal.md %}).

[docker-docs-run]: https://docs.docker.com/engine/reference/run/
[docker-docs-run-detached]: https://docs.docker.com/engine/reference/run/#detached-vs-foreground
[docker-docs-run-clean-up]: https://docs.docker.com/engine/reference/run/#clean-up---rm
[docker-hub-python]: https://github.com/docker-library/python
[docker-docs-ps]: https://docs.docker.com/engine/reference/commandline/ps/
[docker-docs-rename]: https://docs.docker.com/engine/reference/commandline/rename/
[docker-docs-start]: https://docs.docker.com/engine/reference/commandline/start/
[docker-docs-attach]: https://docs.docker.com/engine/reference/commandline/attach/
[docker-docs-exec]: https://docs.docker.com/engine/reference/commandline/exec/

{% include links.md %}
