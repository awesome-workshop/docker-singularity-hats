---
title: "File I/O with Containers"
teaching: 5
exercises: 5
questions:
- "How do containers interact with my local file system?"
objectives:
- "Learn how to copy files to and from a container"
- "Understand when and how to mount a file/volume inside a container"
keypoints:
- "Copy files to an from a container using `docker cp`"
- "Mount a folder/file inside a container using `-v <host path>:<container path>`"
---

<figure align="right" style="display: table; margin: 0; text-align: center;">
<iframe scrolling="no"  src="https://videos.cern.ch/video/OPEN-VIDEO-2021-119-001" width="560" height="315" frameborder="0" allowfullscreen></iframe>
<br/>
<figcaption style="display: table-caption; caption-side: bottom; background: pink; padding: 10px;">Recording of the HATS@LPC2020 session (<a href="https://videos.cern.ch/video/OPEN-VIDEO-2021-119-001">link</a>). Note, you must have a CERN login to watch this video.</figcaption>
</figure>

# Copying

[Copying][docker-docs-cp] files between the local host and Docker containers is possible. On your local host find a file that you want to transfer to the container and then

~~~bash
touch io_example.txt
# If on Mac need to do: chmod a+w io_example.txt
echo "This was written on local host" > io_example.txt
docker cp io_example.txt <NAME>:<remote path>
~~~
{: .source}

**Note:** Remember to do `docker ps` if you don't know the name of your container.

From the container check and modify the file in some way

~~~bash
pwd
ls
cat io_example.txt
echo "This was written inside Docker" >> io_example.txt
~~~
{: .source}

~~~
<remote path>
io_example.txt
This was written on local host
~~~
{: .output}

and then on the local host copy the file out of the container

~~~bash
docker cp <NAME>:<remote path>/io_example.txt .
~~~
{: .source}

and verify if you want that the file has been modified as you wanted

~~~bash
cat io_example.txt
~~~
{: .source}

~~~
This was written on local host
This was written inside Docker
~~~
{: .output}

# Volume mounting

What is more common and arguably more useful is to [mount volumes][docker-docs-volumes] to containers with the `-v` flag. This allows for direct access to the host file system inside of the container and for container processes to write directly to the host file system.

~~~bash
docker run -v <path on host>:<path in container> <image>
~~~
{: .source}

For example, to mount your current working directory on your local machine to the `data` directory in the example container

~~~bash
docker run --rm -it -v $PWD:/home/`whoami`/data sl:7
~~~
{: .source}

From inside the container you can `ls` to see the contents of your directory on your local machine

~~~bash
ls
~~~
{: .source}

and yet you are still inside the container

~~~bash
pwd
~~~
{: .source}

~~~
/home/<username>/data
~~~
{: .output}

You can also see that any files created in this path in the container persist upon exit

~~~bash
touch created_inside.txt
exit
ls *.txt
~~~
{: .source}

~~~
created_inside.txt
~~~
{: .output}

This I/O allows for Docker images to be used for specific tasks that may be difficult to do with the tools or software installed on the local host machine.
For example, debugging problems with software that arise on cross-platform software, or even just having a specific version of software perform a task (e.g., using Python 2 when you don't want it on your machine, or using a specific release of [TeX Live][Tex-Live-image] when you aren't ready to update your system release).

> ## Mounts in Cygwin
> Special care needs to be taken when using Cygwin and trying to mount directories. Assuming you have Cygwin installed at `C:\cygwin` and you want to mount your current working directory:
> ~~~bash
> echo $PWD
> ~~~
> {: .source}
> 
> ~~~
> /home/<username>/<path_to_cwd>
> ~~~
> {: .output}
> 
> You will then need to mount that folder using `-v /c/cygwin/home/<username>/<path_to_cwd>:/home/docker/data`
{: .callout}

> ## `--volume (-v)` versus `--mount`
> 
> The [Docker documentation][docker-docs-bind-mounts] has a full and very interesting discussion about bind/volume mounting using these two options. However, much of it boils down to `--mount` being a more explicit and customizable command. The `-v` syntax combines many of the options found in `--mount` into a single field.
> 
> > "Tip: New users should use the --mount syntax. Experienced users may be more familiar with the -v or --volume syntax, but are encouraged to use --mount, because research has shown it to be easier to use."
> {: .challenge}
> 
> **Key difference:** If a file/directory doesn't exist on the host:
> * `-v` or `--volume` will create the endpoint for you as a directory.
> * `--mount` will generate an error.
{: .callout}

<!--# Running Jupyter from a Docker Container-->
<!---->
<!--You can run a Jupyter server from inside of your Docker container.-->
<!--First run a container while [exposing][docker-docs-run-expose-ports] the container's-->
<!--internal port `8888` with the `-p` flag-->
<!---->
<!--~~~-->
<!--docker run --rm -it -p 8888:8888 matthewfeickert/intro-to-docker /bin/bash-->
<!--~~~-->
<!--{: .source}-->
<!---->
<!--Then [start a Jupyter server][jupyter-docs-server] with the server listening on all IPs-->
<!---->
<!--~~~-->
<!--jupyter notebook --allow-root --no-browser --ip 0.0.0.0-->
<!--~~~-->
<!--{: .source}-->
<!---->
<!--though for your convince the example container has been configured with these default-->
<!--settings so you can just run-->
<!---->
<!--~~~-->
<!--jupyter notebook-->
<!--~~~-->
<!--{: .source}-->
<!---->
<!--Finally, copy and paste the following with the generated token from the server as-->
<!--`<token>` into your web browser on your local host machine-->
<!---->
<!--~~~-->
<!--http://localhost:8888/?token=<token>-->
<!--~~~-->
<!--{: .source}-->
<!---->
<!--You now have access to Jupyter running on your Docker container.-->
<!---->
[docker-docs-cp]: https://docs.docker.com/engine/reference/commandline/cp/
[docker-docs-volumes]: https://docs.docker.com/storage/volumes/
[docker-docs-bind-mounts]: https://docs.docker.com/storage/bind-mounts/
[Tex-Live-image]: https://hub.docker.com/r/matthewfeickert/latex-docker/
[docker-docs-run-expose-ports]: https://docs.docker.com/engine/reference/run/#expose-incoming-ports
[jupyter-docs-server]: https://jupyter.readthedocs.io/en/latest/running.html#starting-the-notebook-server

{% include links.md %}
