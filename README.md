1.  [Setup](#setup)
1.  [Theory](https://github.com/s-mrb/Docker-Notes#theory)
    1.  [Docker](https://github.com/s-mrb/Docker-Notes#docker)
    1.  [Container](https://github.com/s-mrb/Docker-Notes#container)
    1.  [Image](https://github.com/s-mrb/Docker-Notes#image)
    1.  [Dockerfile](https://github.com/s-mrb/Docker-Notes#dockerfile)
    1.  [Base Image](https://github.com/s-mrb/Docker-Notes#base-image)
    1.  [Why we need base image?](https://github.com/s-mrb/Docker-Notes#why-we-need-base-image)
    1.  [If containers don't contain OS then how they are able to run application?](https://github.com/s-mrb/Docker-Notes#if-containers-dont-contain-os-then-how-they-are-able-to-run-application)
    1.  [Docker Registry](https://github.com/s-mrb/Docker-Notes#docker-registry)
    1.  [Docker Daemon](https://github.com/s-mrb/Docker-Notes#docker-daemon)
    1.  [Docker Client](https://github.com/s-mrb/Docker-Notes#docker-client)
    1.  [Data Storage Options in Docker](https://github.com/s-mrb/Docker-Notes#data-storage-options-in-docker)
        1.  [Volumes](https://github.com/s-mrb/Docker-Notes#volumes)
        1.  [Bind Mounts](https://github.com/s-mrb/Docker-Notes#bind-mounts)
        1.  [tmpfs](https://github.com/s-mrb/Docker-Notes#tmpfs)
        1.  [named pipes](https://github.com/s-mrb/Docker-Notes#named-pipes)
        1.  [Volume Type Comparison](https://github.com/s-mrb/Docker-Notes#named-pipes)
    1.  [Copy on Write](https://github.com/s-mrb/Docker-Notes#copy-on-write)
1.  [Docker Network Drivers](https://github.com/s-mrb/Docker-Notes#named-pipes)
    1.  [Bridge](https://github.com/s-mrb/Docker-Notes#bridge)
    1.  [Overlay](https://github.com/s-mrb/Docker-Notes#overlay)
    1.  [Macvlan](https://github.com/s-mrb/Docker-Notes#macvlan)
1.  [Dockerfile](https://github.com/s-mrb/Docker-Notes#dockerfile-1)
    1.  [Common instructions](https://github.com/s-mrb/Docker-Notes#common-instructions)
    2.  [How to make and run dockerfile?](#how-to-make-and-run-dockerfile?)
2.  [Docker Compose](https://github.com/s-mrb/Docker-Notes#docker-compose)
3.  [Docker CLI Commands](https://github.com/s-mrb/Docker-Notes#docker-cli-commands)
    1.  [build](https://github.com/s-mrb/Docker-Notes#build)
    2.  [run](https://github.com/s-mrb/Docker-Notes#run)
    3.  [ps](https://github.com/s-mrb/Docker-Notes#ps)
    4.  [image ls](https://github.com/s-mrb/Docker-Notes#image-ls)
    5.  [stop](https://github.com/s-mrb/Docker-Notes#stop)
    6.  [rm](https://github.com/s-mrb/Docker-Notes#rm)
    7.  [prune](https://github.com/s-mrb/Docker-Notes#prune)
    8.  [push](https://github.com/s-mrb/Docker-Notes#push)
    9.  [pull](https://github.com/s-mrb/Docker-Notes#pull)
    10. [login](https://github.com/s-mrb/Docker-Notes#login)
    11. [tag](https://github.com/s-mrb/Docker-Notes#tag)
    12. [volume create](https://github.com/s-mrb/Docker-Notes#volume-create)
    13. [volume prune](https://github.com/s-mrb/Docker-Notes#volume-prune)
    14. [volume inspect](https://github.com/s-mrb/Docker-Notes#volume-inspect)
    15. [exec](https://github.com/s-mrb/Docker-Notes#exec)
4.  [How to do(s)?](https://github.com/s-mrb/Docker-Notes#how-to-dos)
5.  [MISC](#misc)
6.  [Practice](https://github.com/s-mrb/Docker-Notes#practice)
7.  [Read More](https://github.com/s-mrb/Docker-Notes#read-more)
8.  [Remember](https://github.com/s-mrb/Docker-Notes#remember)
9.  [Internals](https://github.com/s-mrb/Docker-Notes#internals)


## Setup

### Manage Docker as a non-root user

The Docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user `root` and other users can only access it using `sudo`. The Docker daemon always runs as the `root` user.

If you don’t want to preface the `docker` command with `sudo`, create a Unix group called `docker` and add users to it. When the Docker daemon starts, it creates a Unix socket accessible by members of the `docker` group.

> The docker group grants privileges equivalent to the root user. For details on how this impacts security in your system, see Docker Daemon Attack Surface.

To create the docker group and add your user:

1. Create the docker group.
```shell
 sudo groupadd docker
```

2. Add your user to the docker group.
```shell
 sudo usermod -aG docker $USER
```

3. Log out and log back in so that your group membership is re-evaluated.
```shell
newgrp docker 
```
4. Verify that you can run docker commands without sudo.
```shell
docker run hello-world
```


---
---

## Theory

### Docker

Docker is an open source containerization **platform**. It enables developers to **package applications into containers**—standardized executable components combining application source code with the operating system (OS) libraries and dependencies required to run that code in any environment.

---

### Container

- contains a **filesystem** which is isolated from host filesystem.
- contains your actual **application**
- contains all the **dependencies** of your application
- simply **another process on your machine that has been isolated from all other processes on the host machine**. That isolation leverages [kernel namespaces and cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504).
- created from docker `image`, **running an `image` creates `container`**

> **Note:** `container` can have bash-shell, vim-editor etc but does not contain OS.

---

### Image

- The blueprints of our application which form the basis of containers.
- Image is created when you **build `Dockerfile`**

> **Note:** `container` relates to `image` in the same way as `object` relates to `class` in Object Oriented Languages.

---


### Docker Images vs. Containers

In Dockerland, there are images and there are containers. The two are closely related, but distinct. 
For me, grasping this dichotomy has clarified Docker immensely.

#### What's an Image?

An image is an inert, immutable, file that's essentially a snapshot of a container. 
Images are created with the build command, and they'll produce a container when started with run. 
Images are stored in a Docker registry such as registry.hub.docker.com. Because they can become quite large, 
images are designed to be composed of layers of other images, allowing a miminal amount of data to be sent 
when transferring images over the network.

Local images can be listed by running docker images:

```
REPOSITORY                TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                    13.10               5e019ab7bf6d        2 months ago        180 MB
ubuntu                    14.04               99ec81b80c55        2 months ago        266 MB
ubuntu                    latest              99ec81b80c55        2 months ago        266 MB
ubuntu                    trusty              99ec81b80c55        2 months ago        266 MB
<none>                    <none>              4ab0d9120985        3 months ago        486.5 MB
```

#### Important Points:

- IMAGE ID is the first 12 characters of the true identifier for an image. You can create many tags of a given image, but their IDs will all be the same (as above).
- VIRTUAL SIZE is virtual because it's adding up the sizes of all the distinct underlying layers. This means that the sum of all the values in that column is probably much larger than the disk space used by all of those images.
- The value in the REPOSITORY column comes from the -t flag of the docker build command, or from docker tag-ing an existing image. You're free to tag images using a nomenclature that makes sense to you, but know that docker will use the tag as the registry location in a docker push or docker pull.
- The full form of a tag is [REGISTRYHOST/][USERNAME/]NAME[:TAG]. For ubuntu above, REGISTRYHOST is inferred to be registry.hub.docker.com. So if you plan on storing your image called my-application in a registry at docker.example.com, you should tag that image docker.example.com/my-application.
- The TAG column is just the [:TAG] part of the full tag. This is unfortunate terminology.
- The latest tag is not magical, it's simply the default tag when you don't specify a tag.
- You can have untagged images only identifiable by their IMAGE IDs. These will get the <none> TAG and REPOSITORY. It's easy to forget about them.



#### What's a container?

To use a programming metaphor, if an image is a class, then a container is an instance of a class—a runtime object. Containers are hopefully why you're using Docker; they're lightweight and portable encapsulations of an environment in which to run applications.

View local running containers with docker ps:

```
CONTAINER ID        IMAGE                               COMMAND                CREATED             STATUS              PORTS                    NAMES
f2ff1af05450        samalba/docker-registry:latest      /bin/sh -c 'exec doc   4 months ago        Up 12 weeks         0.0.0.0:5000->5000/tcp   docker-registry
```

Here I'm running a dockerized version of the docker registry, so that I have a private place to store my images. Again, some things to note:

- Like IMAGE ID, CONTAINER ID is the true identifier for the container. It has the same form, but it identifies a different kind of object.
- docker ps only outputs running containers. You can view all containers (running or stopped) with  docker ps -a.
- NAMES can be used to identify a started container via the --name flag.

## How to avoid image and container buildup?

One of my early frustrations with Docker was the seemingly constant buildup of untagged images and stopped containers. On a handful of occassions this buildup resulted in maxed out hard drives slowing down my laptop or halting my automated build pipeline. Talk about "containers everywhere"!

We can remove all untagged images by combining docker rmi with the recent dangling=true query:

```
docker images -q --filter "dangling=true" | xargs docker rmi
```

Docker won't be able to remove images that are behind existing containers, so you may have to remove stopped containers with docker rm first:

```
docker rm `docker ps --no-trunc -aq`
```

These are known pain points with Docker, and may be addressed in future releases. However, with a clear understanding of images and containers, these situations can be avoided with a couple of practices:

- Always remove a useless, stopped container with docker rm [CONTAINER_ID].
- Always remove the image behind a useless, stopped container with docker rmi [IMAGE_ID].


---

### Dockerfile

- The text file that contains a list of commands (instructions), which describes how a Docker image is to be built
- Docker can build images automatically by reading the instructions from a Dockerfile
- Using `docker build` users can create an automated build that executes several command-line instructions in succession.

> **Note:** Dockerfile is blueprint of image and image is blueprint of container.

**</> Example:**  
An image is just a snapshot of file system and dependencies or a specific set of directories of a particular application/software. By snapshot I mean, a copy of just those files which are required to run that piece of software (_for example mysql, redis etc._) with basic configurations in a container environment. 

> When you create a container using an image, a small section of resources from your system are isolated with the help of 
> **namespacing** and **cgroups**, and then the files inside the image are copied in this isolated environment of resources.

**Let us understand what is a base image:**  
Suppose you want an image that runs `redis`. You would need:

- a starting point to create the image.
- resolving dependencies
- command to run your application

`</> dockerfile`

```dockerfile
FROM alpine
RUN apk add --update redis
CMD ["redis-server"]
```

- `FROM` - The FROM instruction initializes a **new build stage** and sets the Base Image for subsequent instructions
- `RUN` - Run any **linux command** to install dependencies
- `CMD` - Specifies the **default command** to run when starting a container from this image.

> **Note:** Alpine is the lightest image that contains files just to run basic commands(for example: ls, cd, apk add inside the container)

`docker build:`

```
    Sending build context to Docker daemon  2.048kB
    Step 1/3 : FROM alpine
     ---> a24bb4013296
    Step 2/3 : RUN apk add --update redis
     ---> Running in 535bfd2d1ff1
    fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
    fetch http://dl-
    cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
    (1/1) Installing redis (5.0.9-r0)
    Executing redis-5.0.9-r0.pre-install
    Executing redis-5.0.9-r0.post-install
    Executing busybox-1.31.1-r16.trigger
    OK: 7 MiB in 15 packages
    Removing intermediate container 535bfd2d1ff1
     ---> 4c288890433b
    Step 3/3 : CMD ["redis-server"]
     ---> Running in 7f01a4da3209
    Removing intermediate container 7f01a4da3209
     ---> fc26d7967402
    Successfully built fc26d7967402
```

> This output shows that in Step 1/3 it takes the base alpine image, in Step 2/3, adds a layer of redis to it and then executes the `redis-server` command in Step 3/3 whenever the container is started. _The `RUN` command is only executed when the image is is build process._

So when you pull an image from docker hub (in our example Alpine), it just has the configurations to run the basic requirements. When you need to add your own requirements and configurations to an image, you create a `Dockerfile` and add dependencies layer by layer on a base image to run it according to your needs.

---

### Base Image

Most container-based development starts with a base image and layers on top of it the necessary libraries, binaries, and configuration files necessary to run an application. The **base image is the starting point for most container-based development workflows**.

Most base images are basic or minimal Linux distributions: Debian, Ubuntu, Redhat, Centos, or Alpine. Developers usually consume these images directly from Docker Hub, or other sources. There are official providers along with a wide variety of other downstream repackagers that layer software to meet customer needs.

> **Note:** When we say _these images are linux distros_ we don't mean actual kernals, we meant _filesystem snapshot of those distros_.

---

### Why we need base image?

**you have to maintain coherence between all these layers**, you cannot base your first image on a moving target (i.e. your writable file-system). So, you need a read-only image that will stay forever the same.

---

### If containers don't contain OS then how they are able to run application?

Containers requires a kernel to be running, as images do not provide their own kernel and are not full operating systems.

When the container is started, the layers of the image are joined together to provide everything an app needs to run. The **Docker daemon configures various namespaces (process, mount, network, user, IPC, etc.) to isolate the container from other processes on the same machine**. That's what provides the look and feel of being a separate VM, even when it's just another process on the machine.

> At the end of the day, a container is simply another process running on the machine. It's just one that brought along its entire environment.

---

### Docker Registry

A registry is a **storage and content delivery system**, holding named Docker images, available in different tagged versions.
`docker pull` actually downloads images from registry.

---

### Docker Daemon

The **background service** running on the host that manages building, running and distributing Docker containers. The daemon is the process that runs in the operating system which clients talk to.

---

### Docker Client

The **command line tool** that allows the user to interact with the daemon. More generally, there can be other forms of clients too - such as Kitematic which provide a GUI to the users.

---

### Data Storage Options in Docker

#### Volumes

Created and managed by Docker. You can create a volume explicitly using the `docker volume create` command, or ~Docker can create a volume during container or service creation~.

- When you create a volume, it is stored within a directory on the Docker host.
- When creating the volume you never specify the path in host where data is to be stored, this is managed by docker itself.
- When you mount the volume into a container, this directory is what is mounted into the container.
- A given volume can be mounted into multiple containers simultaneously.
- When no running container is using a volume, the volume is still available to Docker and is not removed automatically. - - You can remove unused volumes using docker volume prune.

> **Note:** Volumes are similar to the way that bind mounts work, except that volumes are managed by Docker and are isolated from the core functionality of the host machine.

##### Types

- `anonymous`  
  docker will automatically assign randomly unique name (unique within the host).

  ```shell
  docker run -v /path/in/container ...
  ```

- `named`

  ```shell
  docker volume create somevolumename
  docker run -v name:/path/in/container ...
  ```

  You can also specify Docker volumes in your docker-compose.yaml file using the same syntax as the examples above. Here’s an example of a named volume

  ```yaml
  volumes:
    - db_data:/var/lib/mysql
  ```

#### Bind Mounts

- Have limited functionality compared to volumes.
- When you use a bind mount, a **file or directory on the host machine is mounted** into a container.
- The file or directory is referenced by its full path on the host machine.
- The **file or directory does not need to exist on the Docker host already**.
- It is created on demand if it does not yet exist.
- They rely on the host machine’s filesystem having a specific directory structure available.
- You can’t use Docker CLI commands to directly manage bind mounts.

#### tmpfs

A tmpfs mount is not persisted on disk, either on the Docker host or within a container. It can be used by a container during the lifetime of the container, to store **non-persistent state or sensitive information**. For instance, internally, swarm services use tmpfs mounts to mount secrets into a service’s containers.

#### named pipes

An `npipe` mount can be used for **communication between the Docker host and a container**. Common use case is to run a third-party tool inside of a container and connect to the Docker Engine API using a named pipe.

### Volume Type Comparison

|                                              | Named Volumes             | Bind Mounts                   |
| -------------------------------------------- | ------------------------- | ----------------------------- |
| Host Location                                | Docker chooses            | You control                   |
| Mount Example (using `-v`)                   | my-volume:/usr/local/data | /path/to/data:/usr/local/data |
| Populates new volume with container contents | Yes                       | No                            |
| Supports Volume Drivers                      | Yes                       | No                            |



### Copy on Write

Copy-on-write is a strategy of sharing and copying files for maximum efficiency. If a file or directory exists in a lower layer within the image, and another layer (including the writable layer) needs read access to it, it just uses the existing file. **The first time another layer needs to modify the file (when building the image or running the container), the file is copied into that layer and modified.** This minimizes I/O and the size of each of the subsequent layers.

---

## Docker Network Drivers

**Keywords:**

- `Outbound Communication:` going from inside the container to outside the host
- `Inbound Communication:` going from outside the host to inside container
- `container to container:` going from one container to another within the same host
- `NAT:` Network address translation is translation of one IP address into another.
- `DNAT:` Destination address translation, translating public address of container into its private address.
- `SNAT:` Source address translation, translating private address of container into its public address.

### Bridge

Creates a Linux virtual bridge and attaches the container to a bridge port.

![1.png](https://github.com/s-mrb/Docker-Notes/blob/main/files/images/1.png)

#### Basic Networking in Docker / Default Bridge

- container host have **Ethernet Adapter** (lets call it `eth0`)
- `eth0` goes into **IP tables** of linux kernel (lets call it `iptables`), these tables are also used for NAT function
- `iptables` is also connected with **docker bridge** (default docker bridge is called `docker0`)
  - run below command to see docker0 as new network after you have installed docker
  ```shell
  ip a | grep "docker0"
  ```
- it is `docker0` with which one or more container can attach

**Note:**  
When you install docker `docker0` a virtual bridge is created and is also added as rule to `iptables`, sepcifically a `MASQUERADE` rule, `MASQUERADE`s in `iptables` function very much like interace based sourcemap, so any packet sourced from range `172.17.0.0/16` (`docker0` bridge subnet) are going to have their source IP's translated to the outgoing interface IP and hence containers are able to access internet (guess so!).

`sudo iptables -t nat --list` to view `iptables`

```shell
⚝      sudo iptables -t nat --list
[sudo] password for xyz:
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  anywhere            !localhost/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16        anywhere
MASQUERADE  all  --  172.19.0.0/16        anywhere

Chain DOCKER (2 references)
target     prot opt source               destination
RETURN     all  --  anywhere             anywhere
RETURN     all  --  anywhere             anywhere
```

**create a network over deafult brige:**

- chose driver default
- set name of bridge as `appbr0`
- set subset
- name network as `app_net`

```shell
docker network create --driver=bridge -o "com.docker.network.brdige.name=appbr0" --subnet=172.200.0.0/16 app_net
```

**see all networks on docker**
network name should be `app_net` not `appbr0`

```shell
docker network ls
```

**find appbr0 among network interfaces of host**

```shell
ifconfig
```

Among all the interfaces you will see:

```shell
appbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.200.0.1  netmask 255.255.0.0  broadcast 172.200.255.255
        ether 02:42:68:65:78:85  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

- `appbr0` is assigned `1` address in that subnet
  Or you could have used grep but it would return less details.

> **Note:** When you attach a network to any container then you get a new `virtual ethernet interface`. Try `ifconfig` to explore it. This new `veth` is be attached to any docker bridge which in turn is attached with `iptables` and hence our container is able to access internet.

**How we are able to access server inside the container from our browser?**  
A `DNAT` rule added in `iptables` while assigning network to a container helps in forwarding packets sent to `x` port of host to `y` port of `container` where port mapping is `x:y`.
You can run `iptables -t nat --list` after and before assigning network to a container to explore more.

#### Custom bridge

LEFT for now

### Host

Attaches the container to the host network. Host network is accessable to container just as it is to the host

### None

Container has only loopback interface

### Overlay

LEFT

### Macvlan

## LEFT

---

---

## Dockerfile

Recipe for creating image.

### Common instructions

- `From`
  - The FROM instruction initializes a new build stage and sets the Base Image for subsequent instructions.
- `RUN `
  - `RUN` has two forms
    - `RUN <command>`
      - shell form, the command is run in a shell, which by default is `/bin/sh -c` on Linux or `cmd /S /C` on Windows
    - `RUN ["executable", "param1", "param2"]`
      - `exec` form
  - In the shell form you can use a `\` to continue a single RUN instruction onto the next line. For example:
  ```dockerfile
  RUN /bin/bash -c 'source $HOME/.bashrc; \
  echo $HOME'
  ```
- `ENTRYPOINT`

  - allows to specify a command along with the parameters

  ```dockerfile
  ENTRYPOINT echo "Hello, $name"
  ```

- `ADD`

  - helps in copying data into a docker image

  ```dockerfile
  ADD /[source]/[destination]
  ```

- `ENV`

  - provides default values for variables which can be accessed within the container

  ```dockerfile
  ENV <key1>=<value1> \
  <key2>=<value2>
  ```

  - environment variable provided via CLI during run command will override environment variables set in Dockerfile

- `MAINTAINER`

  - declares the author field of the container

  ```dockerfile
  MAINTAINER name
  ```

- `EXPOSE`

  - informs the Docker that the container listens on the specified network ports at runtime. You can specify whether the port listens on TCP or UDP, and the default is TCP if the protocol is not specified.
  - it does not actually publish the port. It functions as a type of documentation between the person who builds the image and the person who runs the container about which ports are to be published.
  - to actually publish the port when running the container, use the `-p` flag on `docker run` to publish and map one or more ports, or the `-P` flag to publish all exposed ports and map them to high-order ports.
  - by default `EXPOSE` assumes TCP, you can also specify UDP

  ```dockerfile
  EXPOSE 80/udp
  ```

  - to expose both on tcp and udp

  ```dockerfile
  EXPOSE 80/udp
  EXPOSE 80/tcp
  ```

- `CMD`
  - the default program that will execute once the container runs
- `ADD`

  - two forms

  ```dockerfile
  ADD [--chown=<user>:<group>] <src>...<dest>

  <!-- for paths containing whitespaces -->
  ADD [--chown=<user>:<group>] ["<src>"..."<dest>"]
  ```

  - the `<src>` path must be inside the context of the build; you can not `ADD ../something /something`, because the first step of a `docker build` is to send the context directory (and dubdirectories) to the daemon
  - if `<src>` is a URL and `<dest>` does not end with a trailing slash , then a file is downloaded from the URL and copied to `<dest>`
  - if `<src>` is a URL and `<dest>` does end with a trailing slash, then the filename is inferred from the URL and the file is downloade to `<dest>/<filename>`.
  - if `<src>` is a directory. the entire contents of the directory are copied, including filesystem metadata.
  - the `--chown` feature is only supported on Dockerfiles used to build Linux containers.

- `COPY`

  - two forms

  ```dockerfile
  COPY [--chown=<user>:<group>] <src>...<dest>

  <!-- for paths containing whitespaces -->
  ADD [--chown=<user>:<group>] ["<src>"..."<dest>"]
  ```

- `VOLUME`

  - creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers
  - value can be JSON array or a plain string with multiple arguments
  - `docker run` initializes newly created volume with any data that exists at the specified location within the base image.

  ```dockerfile
  FROM ubuntu
  RUN mkdir/myvol
  RUN echo "hello there!" > /myvol/greeting
  VOLUME /myvol
  ```

  This Dockerfile results in an image that causes `docker run` to create a new mount point at `/myvol` and copy the `greeting` file into the newly created volume.

- `WORKDIR`

  - sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the `Dockerfile`.
  - if `workdir` doesn't exist, it will be created even if it's not used in any subsequent `Dockerfile` instruction
  - can be used multiple times in the `Dockerfile`
  - if relative path is provided, it will be relative to the path of the previous `WORKDIR` instruction

  output of below snippet will be `a/b/c`

  ```dockerfile
  WORKDIR /a
  WORKDIR b
  WORKDIR c
  RUN pwd
  ```

  - `WORKDIR` instruction can resolve environment variables previously set using `ENV`

- `ARG`

  - defines a variable that users can pass at build time to the builder with the `docker build` command using the `--build-arg <key>=<val>` flag

  ```dockerfile
  ARG <name>[=>default value]
  ```

  - if a user specifies a build argument that was not defined in the Dockerfile, the build outputs a warning


---

### How to make and run dockerfile?

1. make `Dockerfile`

```dockerfile
FROM alpine
CMD ["/bin/sh", "-c", "echo 'It works!'"]
```

2. Build the `dockerfile`

```shell
docker build -t imageName:tag path2dockerfile
```

3. Get the `image id`

```shell
docker run imageID
```

OR

```shell
docker run -it nameORid
```

> NOTE: `docker run -it nameORid hostname` won't `echo` **It workds!**, it will be overriden by `hostname` argument and instead runs this command.
> Unlike CMD commands, ENTRYPOINT commands cannot be ignored or overridden—even when the container runs with command line 
> arguments stated.
  
#### Using `CMD` and `ENTRYPOINT` together

When used together
- the `ENTRYPOINT` instruction can be used to define the executable
- `CMD` to define parameters

**Darwin** dockerfile:

```dockerfile
FROM centos:7
RUN    apt-get update
RUN     apt-get -y install python
COPY ./opt/source code
ENTRYPOINT ["echo", "Hello"]CMD [“Darwin”]
```

build image:
```shell
docker build -t darwin .
```

Output when run **without cli parameters**:
```shell
Hello, Darwin
```

Output when run with parameter

```shell
docker run Darwin User_JDArwin
```

will return the output:

```shell
Hello User_JDArwin
```

> This is because the ENTRYPOINT instructions cannot be ignored, while with CMD, the command line arguments override the instruction.

---
---

## Docker Compose

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application's services. Then, with a single command, you create and start all the services from your configuration. ... Run docker-compose up and Compose starts and runs your entire app.

### Example

```yml
# version of app
version: '3'

# containers are services
services:
  # service name
  mymongo:
    # what image makes mymongo
    image: 'mongo'
    ports:
      - '8000:8000'
  mynode:
    # build image for mynode from dockerfile at location .
    build: .
    ports:
      - '8000:8000'
```

For above example to work the database connection url in app code must reference `mymongo` as hostname i.e.

```js
'mongodb://mymongo:27017/testup'

```

---

---

## Docker CLI Commands

### build

**Usage:**

```shell
`docker build [OPTIONS] PATH | URL | -`
```

**Explanation:**  
The `docker build` command builds Docker `images` from a `Dockerfile` and a “context”. A build’s context is the set of files located in the specified PATH or URL.
The URL parameter can refer to three kinds of resources:

- git repositories
- pre-packaged tarball contexts
- plain text files.

**options:**

- `f`
  - Name of the Dockerfile (Default is 'PATH/Dockerfile')
- `t`
  - Name and optionally a tag in the 'name:tag' format
- `o`
  - Output destination (format: type=local,dest=path)
- `cpu-quota`
  - Limit the CPU CFS (Completely Fair Scheduler) quota
- `cpu-period`
  - Limit the CPU CFS (Completely Fair Scheduler) period

[More](https://docs.docker.com/engine/More/commandline/build/)

---

### run

**Usage:**

```shell
`docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]`
```

**Explanation:**  
When an operator executes docker run, the container process that runs is isolated in that it has its own file system, its own networking, and its own isolated process tree separate from the host.
With the docker `run [OPTIONS]` an operator can add to or override the image defaults set by a developer. And, additionally, operators can override nearly all the defaults set by the Docker runtime itself.

**options:**

- `d`
  - run the container in detached mode (in the background)
  - containers started in detached mode exit when the root process used to run the container exits, unless you also specify the `--rm` option.
  - If you use `-d` with `--rm`, the container is removed when it exits or when the daemon exits, whichever happens first.
- `p <p1>:<p2>`
  - map port <p1> of the host to port <p2> in the container
- `name`
  - if you do not assign a container name with the --name option, then the daemon generates a random string name for you.
- `[:TAG]`

  - specify a version of an image you’d like to run the container with - For example, `docker run ubuntu:14.04`

- `a=[]`
  - Attach to `STDIN`, `STDOUT` and/or `STDERR`
    -t
  - Allocate a pseudo-tty
- `sig-proxy=true`
  - Proxy all received signals to the process (non-TTY mode only)
- `i`
  - Keep STDIN open even if not attached
- `rm`
  - automatically clean up the container and remove the file system when the container exits
- `it`
  - `-it` is short for `--interactive + --tty` when you `docker run` with this command.. it would take you straight inside of the container,, where `-d` is short for `--detach` which means you just run the container and then *detach* from it so basically you run container in the background.. so if you run docker container with`-itd` it would run the`-it` options and detach you from the container, so your container still running in the background even without any default app to run..

[More](https://docs.docker.com/engine/More/run/)

---

### ps

**Usage:**

```shell
`docker ps [OPTIONS]`
```

**Explanation:**  
Returns desired overview selected containers.

**options:**

- `a`
  - Show all containers (default shows just running)
- `f`
  - Filter output based on conditions provided
- `s`
  - displays two different on-disk-sizes for each container
    - The `size` information shows the amount of data (on disk) that is used for the writable layer of each container
    - The `virtual size` is the total amount of disk-space used for the read-only image data used by the container and the writable layer
- `no-trunc`
  - Don't truncate output
- `n`
  - Show n last created containers (includes all states)

[More](https://docs.docker.com/engine/More/commandline/ps/)

---

### image ls

**Usage:**

```shell
docker image ls [OPTIONS] [REPOSITORY[:TAG]]
```

**Explanation:**  
Returns desired overview selected containers.

[More](https://docs.docker.com/engine/More/commandline/image_ls/)

---

### stop

**Usage:**

```shell
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

**Explanation:**  
The main process inside the container will receive `SIGTERM`, and after a grace period, `SIGKILL`.

**options:**

- `t`
  - Seconds to wait for stop before killing it
  - Default 10 s

[More]()

---

### rm

**Usage:**

```shell
 docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

**Explanation:**

**options:**

- `f`
  - Force the removal of a running container (uses SIGKILL)
- `l`
  - Remove the specified link
- `v` - Remove anonymous volumes associated with the container

[More](https://docs.docker.com/engine/More/commandline/rm/)

---

### prune

**Usage:**

```shell
docker container prune [OPTIONS]
```

**Explanation:**  
Removes all stopped containers

[More](https://docs.docker.com/engine/More/commandline/container_prune/)

---

### push

**Usage:**

```shell
docker push [OPTIONS] NAME[:TAG]
```

[More](https://docs.docker.com/engine/More/commandline/push/)

---

### pull

Download data from registry. If you use the default storage driver overlay2, then your Docker images are stored in `/var/lib/docker/overlay2`

---

### login

**Usage:**

```shell
docker login [OPTIONS] [SERVER]
```

**Explanation:**  
Login to a registry.

**options:**

- `u`
- `p`

[More](https://docs.docker.com/engine/More/commandline/login/)

---

### tag

**Usage:**

```shell
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

```

**options:**

[More](https://docs.docker.com/engine/More/commandline/tag/)

---

### volume create

**Usage:**

```shell
docker volume create [OPTIONS] [VOLUME]

```

**options:**

- `name`

[More](https://docs.docker.com/engine/reference/commandline/volume_create/)

---

### volume prune

**Usage:**

```shell
docker volume create [OPTIONS] [VOLUME]
```

**Explanation:**
Remove all unused local volumes

**options:**

- `f`
  - Do not prompt for confirmation

[More](https://docs.docker.com/engine/reference/commandline/volume_prune/)

---

### volume inspect

**Usage:**

```shell
docker volume inspect database_name
```

---

### exec

**Usage:**

```shell
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

**Explanation:**

- runs a new command in a running container.
- The command started using docker exec only runs while the container’s primary process (PID 1) is running, and it is not restarted if the container is restarted.
- COMMAND will run in the default directory of the container.
- If the underlying image has a custom directory specified with the `WORKDIR` directive in its Dockerfile, this will be used instead.

> **Note:** COMMAND should be an executable, a chained or a quoted command will not work. Example: `docker exec -ti my_container "echo a && echo b"` will not work, but `docker exec -ti my_container sh -c "echo a && echo b"` will.

**options:**

[More]()

---

### command

**Usage:**

**Explanation:**

**options:**

[More]()

---

---

### How to do(s)?

#### kill all running containers

```shell
docker kill $(docker ps -q)
```

---

#### delete all stopped containers

```shell
docker rm $(docker ps -a -q)
```

---

---

## MISC

### Difference between RUN, CMD and ENTRYPOINT

> `RUN` executed during building of the image, `CMD` is executed from within the image when it is running
 
All three instructions (`RUN`, `CMD` and `ENTRYPOINT`) can be specified in shell form or exec form    

- `RUN`: executes command(s) in a new layer and creates a new image. E.g., it is often used for installing software packages.
- `ENTRYPOINT`: allows to specify a command along with the parameters.
- `CMD`: runs the default program that we want to execute once the container run. E.g., after making the base image and installing all the dependencies for mysql server we have to run the server inside the container, this `CMD` tells the container the instruction that it should run to starts the mysql server. 

You can override `CMD` and `ENTRYPOINT` when running docker run.

Difference between `CMD` and `ENTRYPOINT` **by example**:

    docker run -it --rm yourcontainer /bin/bash            <-- /bin/bash overrides CMD
                                                           <-- /bin/bash does not override ENTRYPOINT
    docker run -it --rm --entrypoint ls yourcontainer      <-- overrides ENTRYPOINT with ls
    docker run -it --rm --entrypoint ls yourcontainer  -la  <-- overrides ENTRYPOINT with ls and overrides CMD with -la

More on difference between `CMD` and `ENTRYPOINT`:

Argument to `docker run` such as /bin/bash overrides any `CMD` command we wrote in Dockerfile.

`ENTRYPOINT` cannot be overriden at run time with normal commands such as `docker run [args]`. The `args` at the end of `docker run [args]` are provided as arguments to `ENTRYPOINT`. In this way we can create a `container` which is like a normal binary such as `ls`.

So `CMD` can act as default parameters to `ENTRYPOINT` and then we can override the `CMD` args from [args].

`ENTRYPOINT` can be overriden with `--entrypoint`.

---


### How do named volumes work in docker?

I'm struggling to understand how exactly does the named volume work in the following example from docker [docs][1]:

<!-- language: lang-yaml -->

    version: "3"

    services:
      db:
        image: db
        volumes:
          #1
          - data-volume:/var/lib/db
      backup:
        image: backup-service
        volumes:
          #2
          - data-volume:/var/lib/backup/data
    
    volumes:
      data-volume:

**ANS:**

My guess is, that the first occurrence of the named volume *(#1)* **defines** what is contained inside the volume, while subsequent occurrences *(#2)* simply **share** the volume's content with whatever containers they are referenced from. 

Is this guess correct?

  [1]: https://docs.docker.com/compose/compose-file/#volume-configuration-reference



Listing `data-volume:` under the top-level `volumes:` key creates a named volume on the host if it doesn't exist yet. This behaves the following way according to this [source](https://madcoda.com/2016/03/docker-named-volume-explained/)

> 1. If you create a named volume by running a new container from image by `docker run -v my-precious-data:/data imageName`, the data within the image/container under /data will be copied into the named volume.
>
> 2. If you create another container binds to an existing named volume, no files from the new image/container will be copied/overwritten, it will use the existing data inside the named volume.
> 3. They don’t have a docker command to backup / export a named volume. However you can find out the actual location of the file by “docker volume inspect [volume-name]”.

In case the volume is empty and both containers have data in the target directory the first container to be run will mount its data into the volume and the other container will see that data (and not its own). I don't know which container will run first (although I expect it executes from top to bottom) however you can force an order with `depends_on` as shown [here](https://docs.docker.com/compose/compose-file/#dependson) 

------------------- Update

> The depends_on option is ignored when deploying a stack in swarm mode with a version 3 Compose file.

---


### Shell command form vs Executable command form

#### Shell Form
A Dockerfile named `Darwin` that uses the shell command will have the following specifications:

```dockerfile
ENV name Darwin
ENTRYPOINT /bin/echo "Welcome, $name"
```

he output of the `docker run -it Darwin` command will be:

```shell
Welcome, Darwin
```

> This command form invokes the shell to go through validation before returning results, which often leads to performance 
> bottlenecks. As a result, shell forms usually aren’t a preferred method unless there are specific command/environment 
> validation requirements.

#### Executable form
Unlike the shell command type, an instruction written in executable form directly runs the executable binaries, without going through shell validation and processing.

To build a Dockerfile named `Darwin` in `exec` form:

```dockerfile
ENV name Darwin
ENTRYPOINT ["/bin/echo", "Welcome, $name"]
```

the output of the `docker run -it Darwin` command will be returned as:
```shell
Welcome, $name
```

To run `bash` in `exec` form, specify `/bin/bash` as executable, i.e.:

```dockerfile
ENV name Darwin
ENTRYPOINT ["/bin/bash", “-c” "echo Welcome, $name"]
```
---


### RUN vs CMD vs ENTRYPOINT

- **RUN:** Mainly used to build images and install applications and packages. Builds a new layer over an existing image by committing the results.
- **CMD:** Sets default parameters that can be overridden from the Docker Command Line Interface (CLI) when a container is running.
- **ENTRYPOINT:** Default parameters that cannot be overridden when Docker Containers run with CLI parameters.

> Any Docker image must have an ENTRYPOINT or CMD declaration for a container to start


---

---

## Practice

- [PWD Training](https://training.play-with-docker.com)
- [Docker 101](https://www.docker.com/101-tutorial)

---

---

## Read More

- [docker networks](https://binarymaps.com/docker-network/)
- [Docker Community](https://docs.docker.com/get-started/resources/)

---

---

## Remember

- Each container also gets its own "scratch space" to create/update/remove files. Any changes won't be seen in another container, even if they are using the same image.
- By default containers can create, update, and delete files, those changes are lost when the container is removed.
- Volumes are often a better choice than persisting data in a container’s writable layer, because a volume does not increase the size of the containers using it.
- Docker does not support relative paths for mount points inside the container.

---

---

## Internals

- [Creating Containers from Scratch](https://youtu.be/8fi7uSYlOdc)

## To cover later

- `-w` in `docker run`
- `docker logs`
- `docker exec`
