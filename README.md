# docker-basics

Docker is a set of platform as a service products that use OS-level virtualization to deliver software in packages called containers. The software that hosts the containers is called Docker Engine.

## [Docker: Getting Started](https://docs.docker.com/get-started/)

***This is assuming you have downloaded Docker***

**What is a Container?**
A container is a runnable instance of an image that is portable (able to run on any OS) and can be run on local machines, virtual machines, or deployed to the cloud. Containers are isolated from each other and run their own software, binaries, and configurations.
You're basically encapsulating a process in a Linux Container which is less resource intensive than a full on virtual machine.

**What is a Container Image?**
An image is an instance of a container. The image contains the container’s filesystem, it must contain everything needed to run an application - all dependencies, configuration, scripts, binaries, etc. The image also contains other configuration for the container, such as environment variables, a default command to run, and other metadata.

### Starting & Running a Container

1. To create a Container Image you must `cd` into your project directory and create a file called [`Dockerfile`](https://docs.docker.com/engine/reference/build/) to specify the environment variables in the image.

2. Now in the terminal use the `docker build` command to create the Container Image

```bash
docker build -t tag-name .
```

*The `-t` indicates that we want to name/tag our image with the tag `tag-name`. Since we named the image `tag-name` we can refer to that image when we run a container*
*The `.` at the end of the command indicate which directory to look for the `Dockerfile`*

3. We start a container using the `docker run` command and specify the name fo the image we just created

```bash
docker run -dp 3000:3000 tag-name
```

*The `-d` flag mean we are running th container in detached mode (in the background) and the `-p` flag creates a mapping between the host's port 3000 to the container's port 3000[^port-flags].

[^port-flags]:
    You only need to worry about specifying a port mapping if you are planning to use your Container Image to Host a Local Server.

#### Replacing and Removing an Old Container Image

Every time you make changes to your project directory you will have to stop, remove, re-build, and re-run your container image[^xDoubt].

[^xDoubt]:
    This is a lie but if you didn't know that then just keep following along

To stop and remove a container you must first get the container's id with the `docker ps` command and then force remove it (if it is still running) with the `docker rm -f <the-container-id>` command. If you just want to stop a container then use the `docker stop <the -container-id>` command.
*`docker ps` only shows you the ids of containers that are currently running, to see all container use the `-a` flag and to remove all stopped containers use the `docker container prune` command.*

### Docker Hub

To Create a Repo:

1. Sign in to Docker Hub

2. Click the **Create Repository** Button

3. Give it a name (image tag name must match repo name)

To Push the Image:

1. Make sure your image's tag matches the repository's name using the `docker tag` command

```bash
docker tag tag-name YOUR-USER-NAME/tag-name
```

*You also have to be logged into your docker account*

2. and then push it using the `docker push` command

```bash
docker push YOUR-USER-NAME/tag-name
```

## Docker Dev Environment

1. [Overview](https://docs.docker.com/desktop/dev-environments/)

2. [Create a Dev Environment](https://docs.docker.com/desktop/dev-environments/create-dev-env/)

3. [Create a Compose Dev Environment](https://docs.docker.com/desktop/dev-environments/create-compose-dev-env/)

4. [Share your Dev Environment](https://docs.docker.com/desktop/dev-environments/share/)

5. [Specify a Dockerfile or base image](https://docs.docker.com/desktop/dev-environments/specify/)

## [Docker Reference Documentation](https://docs.docker.com/reference/)

### File Formats

- [`Dockerfile`](https://docs.docker.com/engine/reference/build/)

- [`docker-compose.yaml` file](https://docs.docker.com/compose/compose-file/)

### Command-line interfaces (CLIs)

- [Cheatsheet](https://docs.docker.com/get-started/docker_cheatsheet.pdf)

- [Docker CLI](https://docs.docker.com/engine/reference/commandline/cli/)

- [Compose CLI](https://docs.docker.com/compose/reference/)
