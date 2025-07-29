## VMachine

runs multiple business applications safely on a single server.

### Flaws

- every VM requires its own dedicated operating system (OS) as every OS consumes CPU, RAM.
- VMs are slow to boot, and portability isn’t
  great — migrating and moving VM workloads between hypervisors and cloud platforms
  is harder than it needs to be.

## Containers

analogous to the VM. A major differ-
ence is that containers do not require their own full-blown OS

## Kubernetes

is the most popular tool for deploying and managing containerized apps.

```note
A containerized app is an application running as a container.
```

Kubernetes used to use Docker as its default container runtime – the low-level technology
that pulls images and starts and stops containers.

`CRI`- Container Runtime Interface

`Containerd`- is the small specialized part of Docker that does the low-level
tasks of starting and stopping containers.

## Summary

We used to live in a world where every time the business needed a new application we
had to buy a brand-new server. VMware came along and allowed us to drive more value
out of new and existing IT assets. As good as VMware and the VM model is, it’s not
perfect. Following the success of VMware and hypervisors came a newer more efficient
and portable virtualization technology called containers. But containers were initially
hard to implement and were only found in the data centers of web giants that had Linux
kernel engineers on staff. Docker came along and made containers easy and accessible to
the masses.

## 2. Docker

Docker is software that runs on Linux and Windows. It creates, manages, and can even
orchestrate containers.

built from various tools from the
`Moby` open-source project

Docker, Inc - company started out as a platform as a service (PaaS) provider called `dotCloud`

“Docker” comes from a British expression
meaning dock worker — somebody who loads and unloads cargo from ships.

Three things to be aware of when referring to
Docker as a technology:

- The runtime
- The daemon (a.k.a. engine)
- The orchestrator

![Docker](images/layers.png)

### Runtime

operates at the lowest level and is responsible for starting and stopping
containers

Docker implements a tiered runtime architecture with high-level and low-
level runtimes that work together.

`RUNC` : The low-level runtime and is the reference implementation of Open Containers Initiative (OCI) runtime-spec.Its job is to interface with the underlying OS and start and stop containers. Every container on a Docker node was created and started
by an instance of runc.

`Containerd`: The higher-level runtime.This manages the entire container lifecycle including pulling images and managing runc instances.

A typical Docker installation has a single long-running containerd process instructing
runc to start and stop containers. runc is never a long-running process and exits as soon
as a container is started.

### Docker daemon (dockerd)

its above containerd and performs higher-level tasks
such as exposing the Docker API, managing images, managing volumes, managing
networks, and more

A major job of the Docker daemon is to provide an easy-to-use standard interface that
abstracts the lower levels.

### Orchestration

Docker also has native support for managing clusters of nodes running Docker. These clusters are called swarms

### The Open Container Initiative (OCI)

The OCI is a governance council responsible for standardizing the low-level fundamental components of container infrastructure. In particular it focusses on image format and
container runtime

### 3.Installing Docker

Ways to install Docker

- Docker Desktop
  - Windows
  - MacOS
- Multipass
- Server installs on
  - Linux
- Play with Docker

### Docker Desktop

is a desktop app from Docker, Inc. that makes it super-easy to work
with containers.

After installing,it includes Docker Compose and you can even enable a
single-node Kubernetes cluster if you need to learn Kubernetes.

Docker Desktop on Windows can run native Windows containers as well as Linux
containers. Docker Desktop on Mac and Linux can only run Linux containers.

$ docker version shows OS/Arch: linux/amd64 for the Server component. This is
because a default installation assumes you’ll be working with `Linux containers`.

switch to Windows containers by right-clicking the Docker whale icon in
the Windows notifications tray and selecting Switch to Windows containers....

**Docker Desktop on Mac
installs all of the Docker engine components in a lightweight Linux VM**

OS/Arch: for the Server component is linux/amd64 or linux/arm64.

for client component -runs on Mac
OS Darwin kernel - shows as either darwin/amd64 or darwin/arm64.

### 4.The Big Picture

`Ops perspective` - download image, start a new container, log in to
the new container, run a command inside of it, and then destroy it.

`Dev perspective` - We’ll clone some app code
from GitHub, inspect a Dockerfile, containerize the app, run it as a container.

### Ops perspective

When you install Docker, you get two major components:

- The Docker client
- The Docker engine (sometimes called the “Docker daemon”)

In a default Linux installation, the client talks to the daemon via a local `IPC/Unix socket` at `/var/run/docker.sock`

On Windows this happens via a named pipe at `npipe:////./pipe/docker_engine.`

### Images

Docker image is an object that contains an OS filesystem, an application, and all application dependencies

In operations - it’s like a virtual machine template which is a stopped VM

For Developer - think of an image as a class.

If it's a freshly installed Docker host, you’ll have no images and it will look like this.

```note
$ docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
```

Getting images onto your Docker host is called `pulling`.

```note
$ docker pull ubuntu:latest
latest: Pulling from library/ubuntu
dfd64a3b4296: Download complete
6f8fe7bff0be: Download complete
3f5ef9003cef: Download complete
79d0ea7dc1a8: Download complete
docker.io/library/ubuntu:latest
```

Run the docker images command again to see the image you just pulled.

```note
$ docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
ubuntu latest dfd64a3b4296 1 minute ago 106M
```

For now, it’s enough to know that an image contains enough of an operating
system (OS), as well as all the code and dependencies to run whatever application it’s designed for.

Each Image gets its own unique ID. You can refer to images using either ID's or names

### Containers

we have an image pulled locally we can use the docker launch a container from it.

```note
$ docker run -it ubuntu:latest /bin/bash
root@6dc20d508db0:/#
```

-it flags switch your shell into the terminal of the
container.your shell is now inside of the new container!

`docker run` tells Docker to start a new container.

`-it flags` tell Docker to make the container interactive and to attach the current shell to the container’s terminal

`ubuntu:latest` the command tells Docker
that we want the container to be based on the ubuntu:latest image

`/bin/bash` - Finally, it tells Docker which process we want to run inside of the container – a Bash shell.

This means the only long-running process inside of the container is the /bin/bash process.

ctrl + P+Q - exit container without terminating it.

`exec` - You can attach your shell to the terminal of a running container with the docker exec
command

```note
$ docker exec -it vigilant_borg bash
root@6dc20d508db0:/#
```

The format of the docker exec command is:

```note
docker exec <options> <container-
name or container-id> <command/app>
```

Stop the container and kill it using docker stop or docker rm

Congratulations, you’ve just pulled a Docker image, started a container from it, attached
to it, executed a command inside it, stopped it, and deleted it.

### The Dev Perspective

Containers are all about apps, In this section, we’ll clone an app from a Git repo, inspect its Dockerfile, containerize it,
and run it as a container.

Dockerfile is a plain-text document that tells Docker how to build the app and
dependencies into a Docker image.

```note
$ cat Dockerfile
FROM alpine
LABEL maintainer="nigelpoulton@hotmail.com"
RUN apk add --update nodejs nodejs-npm
COPY . /src
WORKDIR /src
RUN npm install
EXPOSE 8080
ENTRYPOINT ["node", "./app.js"]
```

For now, it’s enough to know that each line represents an instruction that Docker uses
to build the app into an image.

`docker build` command to create a new image using the instructions in the
Dockerfile. This example creates a new Docker image called test:latest.

```note
$ docker build -t test:latest .
[+] Building 36.2s (11/11) FINISHED
=> [internal] load .dockerignore 0.0s
=> => transferring context: 2B => [internal] load build definition from Dockerfile 0.0s
0.0s
<Snip>
=> => naming to docker.io/library/test:latest => => unpacking to docker.io/library/test:latest 0.0s
0.7
```

Once the build is complete, check to make sure that the new test:latest image exists
on your host.

```note
$ docker images
REPO TAG IMAGE ID CREATED SIZE
test latest 1ede254e072b 7 seconds ago 154MB
```

Run a container from the image and test the app.

```note
$ docker run -d \
--name web1 \
--publish 8080:8080 \
test:latest
```

You’ve copied some application code from a remote Git repo, built it into a
Docker image, and ran it as a container. We call this “containerizing an app”.
