## Theory

### Docker

Docker is an open source containerization platform. It enables developers to package applications into containers—standardized executable components combining application source code with the operating system (OS) libraries and dependencies required to run that code in any environment.

### Container

- contains a filesystem which is isolated from host filesystem.
- contains your actual application
- contains all the dependencies of your application
- simply another process on your machine that has been isolated from all other processes on the host machine. That isolation leverages [kernel namespaces and cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504).
- created from docker `image`, running an `image` creates `container`

> **Note:** `container` can have bash-shell, vim-editor etc but does not contain OS.

### Image

- The blueprints of our application which form the basis of containers.
- Image is created when you build `Dockerfile`

> **Note:** `container` relates to `image` in the same way as `object` relates to `class` in Object Oriented Languages.

### Dockerfile

- The text file that contains a list of commands (instructions), which describes how a Docker image is to be built
- Docker can build images automatically by reading the instructions from a Dockerfile
- Using docker build users can create an automated build that executes several command-line instructions in succession.

> **Note:** Dockerfile is blueprint of image and image is blueprint of container.

**</> Example:**  
An image is just a snapshot of file system and dependencies or a specific set of directories of a particular application/software. By snapshot I mean, a copy of just those files which are required to run that piece of software (_for example mysql, redis etc._) with basic configurations in a container environment. When you create a container using an image, a small section of resources from your system are isolated with the help of **namespacing** and **cgroups**, and then the files inside the image are copied in this isolated environment of resources.

**Let us understand what is a base image:**  
Suppose you want an image that runs redis. You would need:

- a starting point to create the image.
- resolving dependencies
- command to run your application

`</> dockerfile`

```dockerfile
FROM alpine
RUN apk add --update redis
CMD ["redis-server"]
```

- `FROM` - The FROM instruction initializes a new build stage and sets the Base Image for subsequent instructions
- `RUN` - Run any linux command to install dependencies
- `CMD` - Specifies the default command to run when starting a container from this image.

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

### Base Image

Most container-based development starts with a base image and layers on top of it the necessary libraries, binaries, and configuration files necessary to run an application. The base image is the starting point for most container-based development workflows.

Most base images are basic or minimal Linux distributions: Debian, Ubuntu, Redhat, Centos, or Alpine. Developers usually consume these images directly from Docker Hub, or other sources. There are official providers along with a wide variety of other downstream repackagers that layer software to meet customer needs.

> **Note:** When we say _these images are linux distros_ we don't mean actual kernals, we meant _filesystem snapshot of those distros_.

### Why we need base image?

you have to maintain coherence between all these layers, you cannot base your first image on a moving target (i.e. your writable file-system). So, you need a read-only image that will stay forever the same.

### If containers don't contain OS then how they are able to run application?

Containers requires a kernel to be running, as images do not provide their own kernel and are not full operating systems.

When the container is started, the layers of the image are joined together to provide everything an app needs to run. The Docker daemon configures various namespaces (process, mount, network, user, IPC, etc.) to isolate the container from other processes on the same machine. That's what provides the look and feel of being a separate VM, even when it's just another process on the machine.

At the end of the day, a container is simply another process running on the machine. It's just one that brought along its entire environment.

### Docker Registry

A registry is a storage and content delivery system, holding named Docker images, available in different tagged versions

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
  `-` db_data:/var/lib/mysql
  ```

#### Bind Mounts

- Have limited functionality compared to volumes.
- When you use a bind mount, a file or directory on the host machine is mounted into a container.
- The file or directory is referenced by its full path on the host machine.
- The file or directory does not need to exist on the Docker host already.
- It is created on demand if it does not yet exist.
- They rely on the host machine’s filesystem having a specific directory structure available.
- You can’t use Docker CLI commands to directly manage bind mounts.

#### tmpfs

A tmpfs mount is not persisted on disk, either on the Docker host or within a container. It can be used by a container during the lifetime of the container, to store non-persistent state or sensitive information. For instance, internally, swarm services use tmpfs mounts to mount secrets into a service’s containers.

#### named pipes

An npipe mount can be used for communication between the Docker host and a container. Common use case is to run a third-party tool inside of a container and connect to the Docker Engine API using a named pipe.

### Volume Type Comparison

|                                              | Named Volumes             | Bind Mounts                   |
| -------------------------------------------- | ------------------------- | ----------------------------- |
| Host Location                                | Docker chooses            | You control                   |
| Mount Example (using `-v`)                   | my-volume:/usr/local/data | /path/to/data:/usr/local/data |
| Populates new volume with container contents | Yes                       | No                            |
| Supports Volume Drivers                      | Yes                       | No                            |

---

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

- container host have Ethernet Adapter (lets call it `eth0`)
- `eth0` goes into IP tables of linux kernel (lets call it `iptables`), these tables are also used for NAT function
- `iptables` is also connected with docker bridger (default docker bridge is called `docker0`)
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

## Dockerfile

---

---

## Docker Compose

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

- a=[]
  - Attach to `STDIN`, `STDOUT` and/or `STDERR`
    -t
  - Allocate a pseudo-tty
- sig-proxy=true
  - Proxy all received signals to the process (non-TTY mode only)
- i
  - Keep STDIN open even if not attached

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

## Practice

- [PWD](https://training.play-with-docker.com)

---

---

## Read More

- [docker networks](https://binarymaps.com/docker-network/)

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
