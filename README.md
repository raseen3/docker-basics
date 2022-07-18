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

### [Data Persistence](https://docs.docker.com/storage/)

If you find it tedious to keep stopping, removing, building, and running an image every time you make a change you can mount a file path to your container image so that data changes persist even while the container is running.

#### [Volumes](https://docs.docker.com/storage/volumes/)

**Volumes:** provide the ability to connect specific filesystem paths of the container back to the host machine. If a directory in the container is mounted, changes in that directory are also seen on the host machine. If we mount that same directory across container restarts, we’d see the same files.

Create a named volume with `docker volume create my-vol` which creates a volume named `my-vol`.
You can see a list of volumes with `docker volume ls` and you can inspect a volume with `docker volume inspect my-vol` (if you are inspecting a volume named `my-vol`).
To Remove a volume use `docker volume rm my-vol` to remove the volume named `my-vol`.

Use the `-v` flag to declare a volume. The `-v` takes 2 fields[^volume-fields] separated by colon characters (`:`)

[^volume-fields]:
    can take 3 fields but the third field is optional

- first field being the named volume (for anonymous volumes you omit the name)

- the second field is the path where the file or directory are mounted in the container

Example:

```bash
docker build -t volume-container
docker run -v my-vol:/etc/todos volume-container
```

This builds a container and creates a volume and then runs that container with that volume mounted[^docker-volume-create].

[^docker-volume-create]:
    The `docker volume create` command is optional, within the `docker run` command docker recognizes we want to use a named volume and creates one for us automatically.

#### [Bind Mounts](https://docs.docker.com/storage/bind-mounts/)

With **bind mounts**, we control the exact mountpoint on the host. We can use this to persist data, but it’s often used to provide additional data into containers. When working on an application, we can use a bind mount to mount our source code into the container to let it see code changes, respond, and let us see the changes right away.

```bash
docker run -dp 3000:3000 \
     -w /app -v "$(pwd):/app" \
     node:12-alpine \
     sh -c "yarn install && yarn run dev"
```

- `-dp 3000:3000` - same as before. Run in detached (background) mode and create a port mapping

- `-w /app` - sets the "working directory"or the current directory that the command will run from

- `-v "$(pwd):/app"` - bind mount the current directory from the host in the container into the `/app` directory

- `node:12-alpine` - the image to use. Note that this is the base image for our app from the Dockerfile

- `sh -c "yarn install && yarn run dev"` - the command. We’re starting a shell using `sh` (alpine doesn’t have `bash`) and running `yarn install` to install *all* dependencies and then running `yarn run dev`. If we look in the `package.json`, we’ll see that the `dev` script is starting `nodemon`.

Watch the logs using `docker logs -f <container-id>`

#### Volumes versus Bind Mount

Generally Volumes are better with data persistence in that they are less resource intensive but if you are trying to persist code changes then Bind Mount is the way to go.

### [Networking](https://docs.docker.com/get-started/07_multi_container/)

If you want multiple containers to interact with each other's processes then use a Network to help them communicate.

THere are two ways to put a container on a network:

1. Assign it at Start or

2. Connect it to an existing container

Create a network with the `docker network create my-network` command

```bash
docker run -d \
     --network my-network --network-alias mysql \
     -v todo-mysql-data:/var/lib/mysql \
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \
     mysql:5.7
```

starts a container and attaches it to the network `my-network` and because we added `--network-alias mysql` Docker was able to resolve the IP address to the Hostname `mysql`.
This is useful because then we can access one container from another by using the hostname as an environment variable.

### [Docker Compose](https://docs.docker.com/compose/compose-file/)

Docker Compose is a tool that helps manages building and setting up a singular or multiple container so that you only need to type 1 command to start all the containers. The command `docker-compose up` will set everything up based on what is specified in the `docker-compose.yaml` file. `docker-compose down` will tear it all down ([volumes](#volumes) won't be deleted because of persistence)

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
