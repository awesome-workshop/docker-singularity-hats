---
title: "Using Buildah and Podman"
teaching: 30
exercises: 30
questions:
- "Why should I use Buildah and Podman?"
- "Are these cross-platform solutions?"
- "What limitation or extras do Buildah and Podman posses compared to Moby BuildKit and Docker?"
objectives:
- "Be able to choose for yourself whether Buildah and Podman are right for your workflow."
- "Learn how to use Podman and Buildah as replacements for Docker."
keypoints:
- ""
- ""
---

This is the world of containers as you now know it ...

<div id="logos" style="width: 100%;">
	<svg class="logo" style="height: 200px;" fill="rgb(33, 139, 234)" viewBox="-5.724 -43.601 2000 600">
		<path d="m934.957 149.221c-10.479-.345-19.253 7.869-19.599 18.348-.014.417-.014.834 0 1.251v94.24c-45.47-36.517-111.933-29.258-148.449 16.211-15.064 18.758-23.271 42.096-23.264 66.153-.996 58.292 45.451 106.354 103.744 107.351 58.292.996 106.354-45.451 107.351-103.743.021-1.203.021-2.405 0-3.607v-176.512c.157-5.29-1.933-10.399-5.753-14.061-3.741-3.658-8.799-5.655-14.03-5.538m-24.799 221.525c-6.741 15.855-19.321 28.513-35.136 35.352-16.569 7.015-35.274 7.015-51.844 0-15.782-6.775-28.314-19.418-34.951-35.26-6.935-16.383-6.935-34.876 0-51.258 6.651-15.789 19.187-28.368 34.951-35.075 16.569-7.015 35.274-7.015 51.844 0 15.814 6.839 28.395 19.495 35.136 35.352 6.933 16.319 6.933 34.755 0 51.074"></path>
		<path d="m1155.344 270.69c-41.243-41.206-108.082-41.176-149.288.067-19.769 19.785-30.876 46.606-30.886 74.574-.996 58.293 45.451 106.355 103.744 107.352 58.292.996 106.354-45.452 107.351-103.744.021-1.202.021-2.404 0-3.607.009-13.882-2.645-27.637-7.815-40.521-5.326-12.813-13.185-24.418-23.106-34.121m-13.63 100.056c-3.371 7.823-8.17 14.95-14.153 21.015-6.036 6.063-13.166 10.929-21.015 14.337-16.558 7.017-35.254 7.017-51.812 0-15.8-6.76-28.347-19.406-34.982-35.259-6.935-16.383-6.935-34.876 0-51.259 6.67-15.765 19.218-28.312 34.982-34.982 16.558-7.017 35.254-7.017 51.812 0 7.849 3.408 14.979 8.273 21.015 14.338 5.983 6.063 10.782 13.19 14.153 21.014 6.893 16.327 6.893 34.747 0 51.074"></path>
		<path d="m1591.747 259.368c.013-2.595-.521-5.163-1.568-7.538-1.066-2.279-2.513-4.36-4.277-6.154-1.742-1.816-3.836-3.261-6.153-4.246-2.448-1.01-5.073-1.522-7.723-1.508-3.742-.025-7.411 1.044-10.554 3.077l-112.7 73.319v-147.221c.1-5.27-1.982-10.347-5.753-14.03-3.626-3.81-8.68-5.929-13.938-5.846-10.857-.068-19.715 8.679-19.783 19.537v.247 261.983c-.063 5.236 2.016 10.271 5.754 13.938 3.653 3.823 8.741 5.943 14.029 5.846 5.241.085 10.276-2.037 13.876-5.846 3.738-3.667 5.817-8.702 5.754-13.938v-67.934l22.983-15.076 87.103 97.81c3.528 3.393 8.275 5.223 13.168 5.076 2.65.03 5.278-.483 7.723-1.508 2.322-.962 4.418-2.397 6.153-4.215 1.792-1.841 3.241-3.987 4.276-6.338 1.048-2.375 1.582-4.943 1.57-7.538.012-5.112-1.938-10.035-5.446-13.753l-80.979-91.226 78.949-51.258c5.099-3.523 7.963-9.475 7.536-15.66"></path>
		<path d="m1264.752 298.505c6.085-6.016 13.258-10.818 21.138-14.153 8.16-3.472 16.945-5.23 25.813-5.169 7.851-.072 15.646 1.326 22.983 4.123 7.353 2.957 14.163 7.116 20.152 12.307 3.641 2.9 8.176 4.445 12.83 4.369 5.304.223 10.459-1.786 14.215-5.538 3.697-3.768 5.707-8.876 5.568-14.153.065-5.729-2.428-11.188-6.8-14.892-18.885-16.903-43.454-26.056-68.796-25.629-58.301 0-105.562 47.262-105.562 105.562-.146 58.216 46.929 105.527 105.145 105.674 25.412.063 49.992-9.056 69.214-25.679 3.965-3.773 6.192-9.019 6.153-14.491.346-10.479-7.868-19.253-18.347-19.599-.417-.014-.835-.014-1.252 0-4.485.02-8.849 1.463-12.461 4.123-5.892 5.189-12.65 9.304-19.968 12.152-7.345 2.762-15.137 4.139-22.983 4.062-8.867.062-17.653-1.697-25.813-5.169-7.873-3.346-15.045-8.147-21.137-14.152-25.862-25.708-25.988-67.514-.281-93.376.094-.094.188-.187.281-.28"></path>
		<path d="m1983.262 252.969c-3.813-3.578-8.345-6.304-13.292-8-5.657-2.031-11.532-3.394-17.506-4.061-5.91-.726-11.859-1.096-17.814-1.108-12.084-.037-24.08 2.045-35.443 6.154-11.137 4.061-21.531 9.923-30.768 17.353v-3.938c-.431-10.883-9.603-19.357-20.486-18.927-10.279.406-18.521 8.647-18.927 18.927v171.897c.43 10.884 9.602 19.357 20.485 18.928 10.28-.406 18.521-8.647 18.928-18.928v-85.934c-.059-8.877 1.699-17.673 5.168-25.844 3.318-7.836 8.113-14.96 14.123-20.983 6.047-6.003 13.178-10.806 21.014-14.153 8.172-3.472 16.967-5.23 25.845-5.169 8.825-.087 17.587 1.511 25.813 4.707 2.599 1.251 5.427 1.953 8.308 2.062 2.649.024 5.276-.489 7.723-1.508 2.319-.98 4.413-2.425 6.153-4.245 1.758-1.793 3.194-3.874 4.246-6.153 1.068-2.434 1.614-5.065 1.6-7.723.191-4.952-1.732-9.751-5.291-13.2"></path>
		<path d="m1800.104 304.966c-16.765-39.188-55.187-64.69-97.81-64.919-58.283-.017-105.545 47.217-105.562 105.501v.03c-.011 58.318 47.256 105.604 105.573 105.614 25.317.005 49.795-9.087 68.97-25.619.277-.276.708-.646.77-.738 1.719-1.392 3.222-3.029 4.461-4.861 6.336-9.125 4.076-21.658-5.045-27.998-7.646-4.967-17.698-4.065-24.338 2.185-.646.585-2.492 2.308-2.799 2.554l-.277.246c-5.617 4.777-12.033 8.526-18.953 11.076-7.32 2.58-15.037 3.851-22.798 3.754-7.153.035-14.264-1.108-21.046-3.385-6.603-2.207-12.828-5.413-18.46-9.507-5.612-4.105-10.544-9.068-14.614-14.707-4.158-5.757-7.34-12.16-9.415-18.952h149.253c5.237.119 10.299-1.891 14.029-5.569 3.839-3.637 5.934-8.745 5.754-14.03.12-13.913-2.504-27.714-7.723-40.612m-161.345 21.012c1.972-6.798 5.094-13.208 9.229-18.952 4.095-5.648 9.059-10.612 14.707-14.707 5.712-4.09 12.008-7.295 18.676-9.507 6.747-2.247 13.812-3.39 20.922-3.385 7.07-.008 14.095 1.136 20.799 3.385 13.307 4.422 24.901 12.887 33.167 24.214 4.221 5.737 7.445 12.145 9.538 18.952z"></path>
		<path d="m1915.942 422.343c-7.543.119-13.562 6.331-13.443 13.875s6.332 13.562 13.875 13.443c7.495-.118 13.494-6.254 13.445-13.75-.085-7.578-6.297-13.652-13.875-13.568 0 0-.001 0-.002 0m0 24.398c-5.975.272-11.039-4.352-11.311-10.326-.271-5.976 4.352-11.04 10.327-11.312 5.975-.271 11.039 4.352 11.311 10.327.009.19.013.382.011.573.204 5.723-4.27 10.527-9.992 10.731-.115.005-.23.007-.346.007"></path>
		<path d="m1919.081 436.342v-.185c1.512-.292 2.65-1.544 2.8-3.076.057-1.175-.432-2.311-1.323-3.077-1.445-.765-3.076-1.106-4.707-.984-1.743-.024-3.484.12-5.2.431v13.538h3.077v-5.446h1.477c1.754 0 2.554.646 2.83 2.154.184 1.143.536 2.252 1.047 3.292h3.415c-.53-1.062-.873-2.207-1.016-3.385-.138-1.473-1.088-2.744-2.462-3.292m-3.723-.985h-1.508v-3.908c.583-.069 1.172-.069 1.754 0 1.97 0 2.893.831 2.893 2.062s-1.415 2-3.076 2"></path>
		<path d="m707.494 193.557c-1.938-1.539-20.029-15.199-58.181-15.199-10.074.044-20.127.908-30.061 2.584-7.384-50.612-49.228-75.288-51.104-76.395l-10.245-5.908-6.738 9.723c-8.438 13.061-14.598 27.459-18.214 42.582-6.831 28.891-2.677 56.027 11.999 79.226-17.722 9.876-46.151 12.307-51.904 12.522h-470.679c-12.294.017-22.27 9.952-22.337 22.245-.549 41.234 6.437 82.222 20.614 120.946 16.214 42.521 40.336 73.842 71.719 93.01 35.167 21.537 92.302 33.844 157.067 33.844 29.258.092 58.461-2.556 87.226-7.907 39.986-7.342 78.463-21.318 113.839-41.352 29.149-16.88 55.383-38.354 77.688-63.596 37.29-42.213 59.505-89.226 76.026-131.007h6.584c40.828 0 65.935-16.338 79.78-30.029 9.201-8.732 16.384-19.369 21.045-31.167l2.923-8.553z"></path>
		<path d="m65.995 228.909h63.073c3.042 0 5.507-2.466 5.507-5.507v-56.182c.017-3.042-2.435-5.521-5.476-5.538-.01 0-.021 0-.031 0h-63.073c-3.042 0-5.507 2.466-5.507 5.507v.031 56.181c0 3.042 2.465 5.508 5.507 5.508z"></path>
		<path d="m152.913 228.909h63.073c3.042 0 5.507-2.466 5.507-5.507v-56.182c.017-3.042-2.435-5.521-5.477-5.538-.01 0-.021 0-.031 0h-63.073c-3.059 0-5.538 2.479-5.538 5.538v56.181c.018 3.047 2.492 5.508 5.539 5.508"></path>
		<path d="m241.153 228.909h63.073c3.042 0 5.507-2.466 5.507-5.507v-56.182c.017-3.042-2.435-5.521-5.477-5.538-.01 0-.021 0-.031 0h-63.073c-3.042 0-5.507 2.466-5.507 5.507v.031 56.181c.001 3.042 2.467 5.508 5.508 5.508z"></path>
		<path d="m328.348 228.909h63.073c3.047 0 5.521-2.46 5.538-5.507v-56.182c0-3.059-2.479-5.538-5.538-5.538h-63.073c-3.042 0-5.507 2.466-5.507 5.507v.031 56.181c0 3.042 2.466 5.508 5.507 5.508z"></path>
		<path d="m152.913 148.083h63.073c3.046-.017 5.507-2.492 5.507-5.538v-56.181c0-3.042-2.466-5.507-5.507-5.507h-63.073c-3.046 0-5.521 2.46-5.538 5.507v56.181c.017 3.052 2.486 5.521 5.538 5.538"></path>
		<path d="m241.153 148.083h63.073c3.046-.017 5.507-2.492 5.507-5.538v-56.181c0-3.042-2.466-5.507-5.507-5.507h-63.073c-3.042 0-5.507 2.466-5.507 5.507v56.181c0 3.046 2.461 5.521 5.507 5.538"></path>
		<path d="m328.348 148.083h63.073c3.052-.017 5.521-2.486 5.538-5.538v-56.181c-.017-3.047-2.491-5.507-5.538-5.507h-63.073c-3.042 0-5.507 2.466-5.507 5.507v56.181c0 3.046 2.461 5.521 5.507 5.538"></path>
		<path d="m328.348 67.227h63.073c3.047 0 5.521-2.461 5.538-5.507v-56.213c-.017-3.047-2.491-5.507-5.538-5.507h-63.073c-3.042 0-5.507 2.465-5.507 5.507v56.212c0 3.042 2.466 5.508 5.507 5.508"></path>
		<path d="m416.312 228.909h63.073c3.047 0 5.521-2.46 5.538-5.507v-56.182c0-3.059-2.479-5.538-5.538-5.538h-63.073c-3.041 0-5.507 2.466-5.507 5.507v.031 56.181c0 3.042 2.466 5.508 5.507 5.508"></path>
	</svg>
	<img class="logo" src="http://sylabs.io/wp-content/uploads/2022/03/singularity-logo-round.svg" alt="Singularity" title="Singularity" style="float: right; height: 200px;">
</div>
<br><br><br><br><br><br><br><br><br><br>

<!-- https://www.toptal.com/designers/htmlarrows/arrows/ -->
Build, store, run, delete &#8635;

<br><br>

End of story?

<br><br>

No! I already said there was a whole container tool ecosystem!

<img src="../fig/ContainerLandscape.jpg" alt="ContainerLandscape" style="width:100%">

<br>

Here I will talk to you about two more applications in that ecosystem -- Podman and Buildah, both Red Hat projects.

<div id="logos" style="width: 100%;">
	<img class="logo" src="../fig/podman.svg" alt="Podman" title="Podman" style="float: left;">
	<img class="logo" src="../fig/buildah.png" alt="Buildah" title="Buildah" style="float: right;">
</div>

<br><br><br><br><br><br><br><br><br><br>

But first, let's start to disect how the various Docker components interact as a case study. This will help you to understand what motivates the use of these different applications. The image below shows how the Docker CLI (or desktop application) isn't really the main component of the system. It's the Docker daemon which is controlling everything -- talking to the registries, working with image, communicating with the containers, and talking to the underlying kernel. The CLI is just the users way of communicating with the daemon.

<img src="../fig/DockerUnderTheHood.png" alt="DockerUnderTheHood" style="width:600px">

I'm not going to get into the wisdom or folly of choosing a centralized daemon. I'll just say that some people feel strongly that this was not a wise design choice. A few of those reasons are:

* The daemon is a single point of failue.
* The daemon owns of of the child processes (the running containers).
* A failure of the daemon could mean orphaned processes.
* All Docker operations must be conducted using `root` privileges. Everyone with access to the Docker CLI, even non-root users, can bypass most, if not all, of the security features. Anyone with access to the `docker` group should be treated as the `root` and since the daemon is running as `root`, the user can then mount protected system folders and become `root`.
* Using the `--privleged` option allows the container to run as `root` and if the user breaks out, they will be `root` on the host.

Keep in mind, the person writing this tutorial is not a professional pen-tester nor a systems administrator. The descriptions here are very high level. Much of this information can be found in blog posts, articles, and tutorials. For example, I highly recommend watching:

R. McCune, *Docker: security myths, security legends*, Security BSides London, NCC Group, (July 2016) [https://www.youtube.com/watch?v=uQigvjSXMLw](https://www.youtube.com/watch?v=uQigvjSXMLw).

# Podman

To help alleviate some of these issues, Podman takes a different design approach. Instead of using an intermediary daemon to communicate with the various components, Podman directly interacts with the registries, images, containers, and kernels. One key outcome of this design choice is that Podman can be run both as the `root` user or as a `nonroot` user (default). More on that later. Other than the major design choices, Podman makes a few other changes, like storing the images in a different place.

<img src="../fig/PodmanUnderTheHood.png" alt="PodmanUnderTheHood" style="width:600px">

> ## Special note for MacOS and Windows users
> 
> The one caveat to what I have just described is that on MacOS and Microsoft Windows Podman needs to be run inside a lightweight Linux virtual machine (VM). This is because all containers, that I'm aware of, use a Linux kernel and thus cannot run natively on MacOS or Windows. Docker does something similar using its LinuxKit VM, but is slightly more successful at hidding the VM usage from the casual user.
> 
> The first time the user starts the VM they will need Podman to download an image and do some setup. This is accomplished by using the command:
> ~~~bash
> podman machine init
> ~~~
> {: .source}
> This step doesn't need to be repeated unless you'd like to use a different VM or if the VM is deleted.
> 
> The following commands will allow you to start and/or stop the VM:
> ~~~bash
> podman machine start
> # do something interesting
> podman machine stop
> ~~~
> {: .source}
> 
{: .callout}

The goal when designing Podman was that is could seamlessly -- yes, you heard me, seamlessly -- be dropped in as a replacement for Docker. Therefore, all of the Docker commands you are familiar with should work with Podman. Occasionally something is developed for Docker that isn't ported to Podman right away, but these usually aren't very disruptive unless you are on the bleeding edge of Docker usage. Additionally, some convenience flags have been added to the Podman commands.

For example, a typical command to run a container using Docker would be:

~~~bash
docker run --name <container_name> <image> <command>
~~~
{: .source}

The same command using Podman would be:

~~~bash
podman run --name <container_name> <image> <command>
~~~
{: .source}

> ## Note
> 
> By default Podman stores images and containers in the users home area. To avoid filling up your home area when space constrained, you will want to use the options `--root <path> --runroot <path>` to specify a new path for Podman to use.
> 
{: .callout}

Additionally, both Docker and Podman images are based on the same OCI standard, so they are compatible. Podamn can create containers from Docker images pulled from [Docker Hub][docker-hub] and can push images back there or any other OCI compatible container registry. The local repository of Podman images and containers is kept separate from Docker -- because of the new rootless feature and to be compatible with the OCI standard -- but otherwise works similarly to Docker. Podman even has the capability to push and pull images from the local repository managed by the Docker daemon into its own repository. For example:

~~~bash
podman push <image name> docker-daemon:<image name>:<tag>
podman pull docker-daemon:<image name>:<tag>
~~~


<!-- <Why is rootless so cool> -->

I hope you see that Podman isn't somehow inferior to Docker, but instead takes a different design approach.

# Buildah

So, we just said that Podman is a drop-in replacement for Docker and has image building capabilities. Why then do we need Buildah? Well, in fact Podman uses the same code as Buildah to do its image building, but contains a subset of Buildah's functionality. Buildah, can be used on its own, without Podman and contains a superset of image creation and management features. While you can still use Dockerfiles to tell Buildah what to build, the most powerful way to interact with Buildah is by writting Bash scripts. There are a few additional points you should keep in mind:

1. Buildah gives the user finer control of creating image layers and the ability to commit many changes to a single layer.
2. Buildah's `run` command is like a Dockerfile's `RUN` command, not Docker or Podman's `run` command. This is because Buildah is mean't for building images. Basically, you're telling Buildah how to build an image, not how to run it.
3. Buildah can actually create images from scratch, meaning you start with nothing ... yes. This is useful for creating lightweight images containing only the packages needed in order to run your application.
4. Buildah makes OCI compatible images by default, but it can also produce other image formats.
5. Buildah runs as an unprivileged user. Big win for security!

> ## Fun Fact
>
> Buildah is named because of the way Dan Walsh, a Distinguished Red Hat Developer, pronounces "builder" due to his Boston accent.
> 
{: .testimonial}

A typical set of commands to build and view an image using Docker would be:

~~~bash
docker build -f <dockerfile> -t <tag> <path>
docker images
~~~
{: .source}

The same commands using Buildah would be:

~~~bash
buildah build -f <dockerfile> -t <tag> <path>
buildah images
~~~
{: .source}

> ## Note
> 
> By default Buildah stores the images in the users home area. To avoid filling up your home area when space constrained, you will want to use the options `--root <path> --runroot <path>` to specify a new path for Buildah to use. Typically you will want this to be the same path used by Podman.
> 
{: .callout}

# Running Unprivileged Containers




# Resources

If you would like to read some more articles on the differences between Docker and Podman/Buildah, take a look at:

* [Podman and Buildah for Docker users](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users)
* [Say "Hello" to Buildah, Podman, and Skopeo](https://www.redhat.com/en/blog/say-hello-buildah-podman-and-skopeo)

For more information about what's running under the hood and why containers running on MacOS and Windows need a VM, take a look at:

* [How Podman runs on Macs and other container FAQs](https://www.redhat.com/sysadmin/podman-mac-machine-architecture)
* [Why is Docker on macOS So Much Worse Than Linux?](https://dev.to/ericnograles/why-is-docker-on-macos-so-much-worse-than-linux-flh)

For more information on rootless Podman from the master himself (Dan Walsh), take a look at:

* [How does rootless Podman work?](https://opensource.com/article/19/2/how-does-rootless-podman-work)

{% include links.md %}