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

## Docker CLI Commands


### build

**Usage:**    
- `docker build [OPTIONS] PATH | URL | -`

**Explanation:**    
The `docker build` command builds Docker `images` from a `Dockerfile` and a “context”. A build’s context is the set of files located in the specified PATH or URL. 
The URL parameter can refer to three kinds of resources:
-   git repositories
-   pre-packaged tarball contexts 
-   plain text files.    

**options:**    
-   `f` 
        -   Name of the Dockerfile (Default is 'PATH/Dockerfile')
-   `t` 
        -   Name and optionally a tag in the 'name:tag' format
-   `o` 
        -   Output destination (format: type=local,dest=path)
-   `cpu-quota` 
        -   Limit the CPU CFS (Completely Fair Scheduler) quota
-   `cpu-period` 
        -   Limit the CPU CFS (Completely Fair Scheduler) period





[Reference](https://docs.docker.com/engine/reference/commandline/build/)

### run
**Usage:**    
`docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]`

**Explanation:**    
When an operator executes docker run, the container process that runs is isolated in that it has its own file system, its own networking, and its own isolated process tree separate from the host.
With the docker `run [OPTIONS]` an operator can add to or override the image defaults set by a developer. And, additionally, operators can override nearly all the defaults set by the Docker runtime itself.

**options:**    
-   `d` 
        -   run the container in detached mode (in the background)
        -   containers started in detached mode exit when the root process used to run the container exits, unless you also specify the `--rm` option. If you use `-d` with `--rm`, the container is removed when it exits or when the daemon exits, whichever happens first.
-   `p <p1>:<p2>` 
        -   map port <p1> of the host to port <p2> in the container
-   `name`
        -   if you do not assign a container name with the --name option, then the daemon generates a random string name for you.
-   `[:TAG]`
        -   specify a version of an image you’d like to run the container with
        -   For example, `docker run ubuntu:14.04`

    
    [Reference](https://docs.docker.com/engine/reference/run/)


### ps
**Usage:**    
`docker ps [OPTIONS]`

**Explanation:**    
Returns desired overview selected containers.

**options:**    
-   `a`
        -   Show all containers (default shows just running)
-   `f`
        -   Filter output based on conditions provided
-   `s`
        -   displays two different on-disk-sizes for each containe
                -   The “size” information shows the amount of data (on disk) that is used for the writable layer of each container
                -   The “virtual size” is the total amount of disk-space used for the read-only image data used by the container and the writable layer.
-   `no-trunc`
        -   Don't truncate output
-   `n`
        -   Show n last created containers (includes all states)

[Reference](https://docs.docker.com/engine/reference/commandline/ps/)


NEXT
- ps
- docker stop <the-container-id>
- docker rm <the-container-id>
- docker registry - default docker hub
- docker push <docker id>/<reponame>:<tag>
- docker image ls
- docker login -u YOUR-USER-NAME
- Use the docker tag command to give the docker-101 image a new name. Be sure to swap out YOUR-USER-NAME with your Docker ID
    - docker tag docker-101 YOUR-USER-NAME/101-todo-app









### command
**Usage:**    


**Explanation:**    

**options:**    
    
[Reference]()


## Practice

## Internals

- [Creating Containers from Scratch](https://youtu.be/8fi7uSYlOdc)
