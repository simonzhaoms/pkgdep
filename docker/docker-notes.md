# Docker notes #

## Introduction ##

<details>
<summary>Click to see more ...</summary>

See [Docker Engine overview](https://docs.docker.com/engine/).
* **Docker**
  + is a platform that provides the tools to package and run an
    application in an environment called a **container** that is
    isolated from the host like a hypervisor-based virtual machine
    (such as VirtualBox) but is lightweight.  See also 
    - [Use containers to Build, Share and Run your applications](https://www.docker.com/resources/what-container/)
    - [Docker -- Glossary](https://docs.docker.com/glossary/#docker)
  + uses a client-server arthitecutre:
    - **The Docker daemon** (also called **Docker Engine**)
      * a long-running daemon process
        ([`dockerd`](https://docs.docker.com/engine/reference/commandline/dockerd/))
        listens for Docker API requests and manages Docker objects
        such as images, containers, networks, and volumes.
      * It can also communicate with other daemons to manage Docker
        services.
    - **The Docker client** (also called **Docker CLI client**)
      * a CLI client
        ([`docker`](https://docs.docker.com/engine/reference/commandline/docker/))
        sends commands (such as `docker pull`, `docker run`) to the
        Docker daemon via [**Docker
        API**](https://docs.docker.com/engine/api/).
      * It can communicate with more than one daemon.
  + can be used in the following scenarios:
    - CI/CD workflow
    - Sandboxed local development/testing
      * Since it's lightweight and a contaienr has everything that an
        application requires, running an application is fast and will
        not contaminate the environment on the host.
      * So it's very suitable for testing a tool, or sandboxing any
        applications even like browsers, no matter whatever OS they
        require.  See also [Test -- Build with
        Docker](https://docs.docker.com/build/guide/test/).
      * If you are a Python developer, you usually create a Conda
        environment for your Python project at hand.  Using Docker,
        you can put your environment in a container, and use your
        favorate IDE (such as Visual Studio Code) to interact with it
        like working with a remote server.
    - Export binaries
      * Sometimes, we just want to use Docker container as an isolated
        environment like a virtual machine to compile souce code and
        get the binaries only, instead of a Docker image.  See also
        [Export binaries -- Build with
        Docker](https://docs.docker.com/build/guide/export/).
    - Cross-compilation.
      * [Multi-platform -- Build with Docker](https://docs.docker.com/build/guide/multi-platform/)
      * [Multi-platform images](https://docs.docker.com/build/building/multi-platform/)
      * [xx - Dockerfile cross-compilation helpers](https://github.com/tonistiigi/xx)
* **Docker image**
  + also called container image, is an ordered collection of root
    filesystem changes and the corresponding execution parameters for
    use within a container runtime.  An image typically contains a
    union of layered filesystems stacked on top of each other, and is
    built from a Dockerfile.
* **Docker container**
  + is a runnable instance of an Docker image -- what the Docker image
    becomes in memory when executed (that is, an image with state, or
    a user process).
  + In terms of implementation, the difference between an image and
    its containers is the top writable layer called the container
    layer.  It means that a container is made up of the single top
    writable layer plus the underlying read-only layers from the
    image.  All writes to the container that add new or modify
    existing data are stored in this writable layer.  When the
    container is deleted, the writable layer is also deleted.  The
    underlying image remains unchanged.  The layer filesystem is
    managed by a [storage
    driver](https://docs.docker.com/storage/storagedriver/).  Docker
    uses storage drivers to store image layers, and to store data in
    the writable container layer.  See [Container and
    layers](https://docs.docker.com/storage/storagedriver/#container-and-layers)
    for more details.
  + runs as a sandboxed process containing everything needed to run
    the application, so it can be started and stoped by using the
    Docker API or CLI.
* [**Dockerfile**](https://docs.docker.com/engine/reference/builder/)
  + is a text document that contains all the instrutions to build a
    Docker image.
* [**Docker Compose**](https://docs.docker.com/compose/)
  + is a tool for defining and running containers in a [Compose
    file](https://docs.docker.com/compose/compose-file/) usually
    called `compose.yaml`.
  + is typically helpful when running multi-container applications.
    An application usually contains several parts, such as a frontend,
    and a backend database, running in different containers.  Instead
    of running one-by-one manually, Docker compose helps us define the
    parts and run them all at once.
* [**Swarm mode**](https://docs.docker.com/engine/swarm/)
  + refers to cluster management and orchestration features embedded
    in Docker Engine.  It is similar Kubernetes, but provided natively
    in Docker.
* [**Docker Desktop**](https://docs.docker.com/desktop/)
  + is a bundled application that includes the Docker daemon/Engine,
    the Docker CLI client, Docker Compose, Docker Build, Docker
    Extensions, Docker Content Trust, Kubernetes, and Credential
    Helper.
* **Docker registry**
  + is a place storing Docker images.
* **Docker Hub** (https://hub.docker.com)
  + is a public registry that Docker looks for images by default.

</details>


## Installation on Ubuntu ##

<details>
<summary>Click to see more ...</summary>

There are 2 ways to install Docker (See [Install Docker on
Linux](https://docs.docker.com/desktop/install/linux-install/)):
1. Install [Docker Server/Engine](https://docs.docker.com/engine/).

   ```bash
   ARCH=$(dpkg --print-architecture)
   CODENAME=$(. /etc/os-release && echo "$VERSION_CODENAME")
   
   APT_URL="https://download.docker.com/linux/ubuntu"
   APT_LIST="/etc/apt/sources.list.d/docker.list"
   KEYRING_DIR="/etc/apt/keyrings"
   GPG_PATH="${KEYRING_DIR}/docker.asc"
   GPG_URL="${APT_URL}/gpg"
   
   APT_ENTRY="deb [arch=${ARCH} signed-by=${GPG_PATH}] ${APT_URL} ${CODENAME} stable"
   
   # Update APT package index in case of new fresh install
   sudo apt-get update
   
   # Install necessary dependencies
   sudo apt-get install -y ca-certificates curl
   
   # Add Docker's official GPG key
   sudo install -m 0755 -d "${KEYRING_DIR}"
   curl -fsSL "${GPG_URL}" | sudo gpg --dearmor -o "${GPG_PATH}"
   sudo chmod a+r "${GPG_PATH}"
   
   # Set Docker APT repo source
   echo "${APT_ENTRY}" | sudo tee "${APT_LIST}" > /dev/null
   sudo apt-get update
   
   # Install the latest Docker community edition
   sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   
   # Test if docker is installed properly.
   sudo docker run hello-world
   ```

1. Install [Docker Desktop](https://docs.docker.com/desktop/).
   + Docker Desktop is GA, but it is not recommended to have both
     Docker Desktop and Docker Engine on the same machine.
   + Unlike Docker Engine, Docker Desktop on Linux runs a VM instead
     of directly on the machine.  More details can be found at [FAQs
     for Linux](https://docs.docker.com/desktop/faqs/linuxfaqs/).

</details>


## Workflow ##

<details>
<summary>Click to see more ...</summary>

1. Write a Dockerfile to define the environment containerizing your
   application.  More details can be found at [make a Docker
   image](#make-a-docker-image).
1. Build the Docker image defined in the Dockerfile using [`docker
   image
   build`](https://docs.docker.com/engine/reference/commandline/image_build/)
   (`docker build`).

   ```bash
   docker build -t <IMAGE-TAG> <DOCKERFILE-DIRECTORY>
   ```

   * We can give the Dockerfile another name, and specify it by
     `docker build -f <DOCKERFILE>`.
   * To list available images, use [`docker image
     ls`](https://docs.docker.com/engine/reference/commandline/image_ls/)
     (`docker images`).
     
     ```console
     $ docker images
     REPOSITORY                          TAG       IMAGE ID       CREATED        SIZE
     myproject                           latest    7a5e87002f14   2 days ago     7.38MB
     multi-container-app-main-todo-app   latest    a1bccf51da38   2 days ago     226MB
     <none>                              <none>    c642d7a9236e   2 days ago     226MB
     ubuntu                              latest    3db8720ecbf5   6 days ago     77.9MB
     mongo                               6         af2af2fc1fab   13 days ago    689MB
     bash                                latest    b6281a9c2552   5 weeks ago    14MB
     docker/welcome-to-docker            latest    c1f619b6477e   3 months ago   18.6MB
     hello-world                         latest    d2c94e258dcb   9 months ago   13.3kB
     $ docker images bash
     REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
     bash         latest    b6281a9c2552   5 weeks ago   14MB
     ```

   * To view the layers in the image, use [`docker image
     history`](https://docs.docker.com/engine/reference/commandline/image_history/)
     (`docker history`).
     
1. Create and run a container from the image to test your application
   using [`docker container
   run`](https://docs.docker.com/engine/reference/commandline/container_run/)
   (`docker run`).  More details can be found at [Running
   containers](#running-containers).

   ```bash
   docker run <IMAGE-TAG>
   ```

1. Update the image by modifying the Dockerfile and rebuild it.
1. Stop and remove the container using [`docker container
   stop`](https://docs.docker.com/engine/reference/commandline/container_stop/)
   (`docker stop`) and [`docker container
   rm`](https://docs.docker.com/engine/reference/commandline/container_rm/)
   (`docker rm`).

   ```bash
   docker stop <CONTAINER-ID>
   docker rm <CONTAINER-ID>
   ```

   * By default, a container's file system persists after the
     container exits so that we can inspect the container for
     debugging.  That's why we use `docker rm` to delete the container
     to save spaces.  To let Docker automatically clean up the
     container file system when it exits, we can use [`docker run
     --rm`](https://docs.docker.com/engine/reference/commandline/container_run/#rm)
     when running a container.
   * The 2 commands above can be completed by the single command
     `docker rm -f <CONTAINER-ID>`.
   * Useful commands for removal:
   
     ```bash
     # Remove all containers
     docker rm $(docker ps -aq)
     
     # Remove only those containers stopping running
     docker rm $(docker ps -aq -f "status=exited")
     docker container prune
     ```
 
   * To re-run a stopped container, we can use [`docker container
     start`](https://docs.docker.com/engine/reference/commandline/container_start/)
     (`docker start`).  By using the `-a/--attach` option of `docker
     start`, we can attach to interact with it.

1. Create and run a new container from the new image to test the
   update.
1. Share the image by pushing it to a Docker registry such as Docker
   Hub, using [`docker image
   push`](https://docs.docker.com/engine/reference/commandline/image_push/)
   (`docker push`).

</details>


## Running containers ##

<details>
<summary>Click to see more ...</summary>

See also [Running
containers](https://docs.docker.com/engine/reference/run/).

* [`docker container
  run`](https://docs.docker.com/engine/reference/commandline/container_run/)
  (`docker run`) is used to create and run a contaienr from an Docker
  image.  Instead of re-using previous container of the same image, a
  new container will be created every time you run this command, thus
  every container have a unique ID and name which can be referred to
  afterwards.
* To list avaiable containers, use [`docker container
  ls`](https://docs.docker.com/engine/reference/commandline/container_ls/)
  (`docker ps`).
  
  ```
  $ # -a is used to show all containers including those finishing running,
  $ # otherwise, only running containers are listed.
  $ docker container ls -a
  CONTAINER ID  IMAGE        COMMAND   CREATED       STATUS                   PORTS  NAMES
  caf880f16684  hello-world  "/hello"  16 hours ago  Exited (0) 16 hours ago         eager_cori
  $
  $ # list only container ID
  $ docker container ls -aq
  caf880f16684
  ```

* A name can be given to the container via the `--name` option for
  later reference.
  
  ```console
  $ docker run --name mytest hello-world
  $ docker ps -a
  CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                     PORTS     NAMES
  43218a4b9223   hello-world   "/hello"   6 seconds ago    Exited (0) 6 seconds ago             mytest
  a318b8ca5283   ubuntu        "bash"     35 minutes ago   Up 34 minutes                        trusting_knuth
  $ docker rm mytest
  mytest
  $ docker ps -a
  CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
  a318b8ca5283   ubuntu    "bash"    35 minutes ago   Up 35 minutes             trusting_knuth
  ```

* The container runs in the foreground by default until the process
  finishes.
  + Containers of some images such as the `bash` image will exit
    immediately when running.
    
    ```console
    $ docker ps -a
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
    $ docker run --name mytest bash
    $ docker ps -a
    CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                     PORTS     NAMES
    f6f667f1fbe0   bash      "docker-entrypoint.s…"   3 seconds ago   Exited (0) 2 seconds ago             mytest
    ```
    
    - To prevent the container exiting, we can use the
      `-i/--interactive` option to keep STDIN open, and add together
      the `-t/--tty` option to attach a pseudo-TTY, so that the input
      and output feature (such as echo-off) of TTY devices can be
      used, thus we create an interactive terminal session for the
      container.  See also [Keep STDIN
      open](https://docs.docker.com/engine/reference/commandline/container_run/#interactive)
      and [Allocate a
      pseudo-TTY](https://docs.docker.com/engine/reference/commandline/container_run/#tty).
    - To escape the interactive terminal session but leave the
      container run in the backgroud without exiting, we can type
      `Ctrl + P Ctrl+ Q`.  If we use `Ctrl + D` when in the terminal
      session of the container, the container will exit intead of
      keeping running in the background.
    - To re-attach to the interactive termianl session running in the
      backgroud, we can use [`docker container
      attach`](https://docs.docker.com/engine/reference/commandline/container_attach/)
      (`docker attach`).
    - To make the container running in the background without exiting,
      we can use the `-d/--detach` option toghter with `-it` to make
      it run in detached mode.
    
    ```console
    $ docker run --name mytest -it bash
    bash-5.2# pwd  # Now we are in the interactive terminal session of the contaienr
    /
    bash-5.2#  # Type Ctrl + P Ctrl + Q to escape
    $ docker ps
    CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
    1644696b8cac   bash      "docker-entrypoint.s…"   14 minutes ago   Up 14 minutes             mytest
    $ docker attach mytest
    bash-5.2# pwd
    /
    bash-5.2#  # Type Ctrl + P Ctrl + Q to escape
    $ docker ps
    CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
    1644696b8cac   bash      "docker-entrypoint.s…"   25 minutes ago   Up 25 minutes             mytest
    Simons-MacBook-Pro:~ simon$ docker rm -f mytest
    mytest
    Simons-MacBook-Pro:~ simon$ docker ps
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
    Simons-MacBook-Pro:~ simon$ docker run --name mytest -it -d bash
    dc45b105b4192c04b30476cdf5eaa82cb3b1c87d954fcc4a8d24b07790d1b6d3
    Simons-MacBook-Pro:~ simon$ docker ps
    CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
    dc45b105b419   bash      "docker-entrypoint.s…"   2 seconds ago   Up 2 seconds             mytest
    Simons-MacBook-Pro:~ simon$ docker attach mytest
    bash-5.2# pwd
    /
    bash-5.2# 
    ```

* The `-a/--attach` option can be used to specify which of STDIN,
  STDOUT and STDERR of the contaienr needs to be connected to the
  host.
  
  ```
  # Attach STDIN and STDOUT only
  docker run -a stdin -a stdout <IMAGE-TAG>
  ```

  + It is usually used when containers are piped because the attach is
    transient.  To keep STDIN attached and open all the time, use the
    `-i/--interactive` option instead.
  + See also [Understanding docker run --attach
    option](https://forums.docker.com/t/understanding-docker-run-attach-option/134337/4)
    and [Attach to
    STDIN/STDOUT/STDERR](https://docs.docker.com/engine/reference/commandline/container_run/#attach)
* To view the details of a container, use [`docker container
  inspect`](https://docs.docker.com/engine/reference/commandline/container_inspect/)
  with its ID.
* To view the container logs, use [`docker container
  logs`](https://docs.docker.com/engine/reference/commandline/container_logs/)
  (`docker logs`) with its ID or name.
* To mount a directory on the host to a directory in the container:

  ```bash
  # There is another new recommended option `--mount` for `-v`
  docker run -v <HOST-PATH>:<CONTAINER-PATH> <IMAGE-TAG>
  ```

* To run a command in a running container, use [`docker container
  exec`](https://docs.docker.com/engine/reference/commandline/container_exec/)
  (`docker exec`):

  ```bash
  docker exec <CONTAINER-ID> <COMMAND>
  ```
  
  Use the `-it` option, we can run the command interatcitvely.

* For multi-container applications, networking is required.

  ```bash
  # Create a network
  docker network create <NEWTORK-NAME>

  # Run a container and connnet it to the network using the
  # `--network` option, thus the container will have its own IP.
  # Meanwhile, give a domain name to the container using the
  # `--network-alias` option for later reference by name instead
  # of IP.
  docker run --network <NETWORK-NAME> --network-alias <DOMAIN-NAME1> <IMAGE-TAG1>...

  # Run another container and connect it to the network as well
  docker run --network <NETWORK-NAME> -v <HOST-PATH>:<CONTAINER-PATH> <IMAGE-TAG2>...
  ```

  To simplify the running of multiple containers, Docker Compose can
  be used to spin everything up by defining the operations in a YAML
  file called `compose.yaml` and then running [`docker compose
  up`](https://docs.docker.com/engine/reference/commandline/compose_up/),
  so the two `docker run`s above can be configured as below:

  ```yaml
  services:
    <DOMAIN-NAME1>:
      # No <NETWORK-NAME> is required, since it will be created
      # automatically when using Docker Compose
      image: <IMAGE-TAG1>
    <DOMAIN-NAME2>:
      image: <IMAGE-TAG2>
      volumes:
        - <HOST-PATH>:<CONTAINER-PATH>
  ```

</details>


## Make a Docker image ##

<details>
<summary>Click to see more ...</summary>

There are 2 ways:
1. Write a Dockerfile to build an image.
   * Describe all the commands used to create the image in the
     Dockerfile.
1. Save an existing container into an image.
   * Setup the environment in a container, then commit it into an
     image.

### Via Dockerfile ###

<details>
<summary>Click to see more ...</summary>

1. Write Dockerfile, consisting of instructions in the format of
   `INSTRUCTION arguments`.

   <details>
   <summary>Click to see more ...</summary>

   For example:

   ```dockerfile
   # Specify on which image this image is built, equivalent to:
   #     FROM ubuntu:latest
   # As I know, Dockerfile directives are case-insensitive, so it is also equivalent to
   #     from ubuntu
   # You can also give a specific tag verson, such as:
   #     FROM ubuntu:bionic-20181112
   # You can check available tag version of ubuntu at https://hub.docker.com/_/ubuntu/
   # You can also search for other available images at https://hub.docker.com
   # The size of the container run from the latest ubuntu image is about 86.2MB.
   # In Dockerfile, comments are prefixed by '#'.
   FROM ubuntu

   # Run the commands.  Here we install python3.
   # Make sure to 'apt-get update' before 'apt-get install', and use 'apt-get' instead of 
   # 'apt', otherwise, you will get a warning:
   #     WARNING: apt does not have a stable CLI interface. Use with caution in scripts.
   RUN apt-get update; apt-get install -y python3
   
   # Copy hello.py from your local filesystem into /tmp/ of the filesystem in the image
   COPY hello.py /tmp/
   
   # Set the environment variable NAME as Simon
   ENV NAME Simon
   
   # Set the default application or command on startup when you 'docker run' this image.
   # Here is 'python3 /tmp/hello.py'
   CMD ["python3", "/tmp/hello.py"]
   ```

   * The list of available instrunctions can be found at [Dockerfile
     reference](https://docs.docker.com/engine/reference/builder/#overview).
     See also [Best practices for Dockerfile
     instructions](https://docs.docker.com/develop/develop-images/instructions/).
   * The instruction is case-insensitive.  However, convention is for
     them to be uppercase to distinguish them from arguments more
     eaily.
   * The order of Dockerfile instructions matters because each
     instruction in a Dockerfile roughly translates to an image layer,
     except instructions such as `LABEL` and `CMD` which only modifies
     the image's metadata.
   * When building an image, the Docker builder attempts to reuse
     layers from earlier builds.
     + If a layer of an image is unchnaged, then the builder picks it
       up from the build cache.
     + If a layer has changed since the last build, that layer and all
       following layers must be rebuilt, resulting in slow build.
       - If we change `hello.py` in the Dockerfile example above.  The
         instruction `COPY hello.py /tmp/` and all its following
         instructions will be rebuilt.
   * In a Dockerfile, a `FROM` instruction starts a build stage.
     + Multiple stages are allowed in a Dockerfile.
     + Stages are indexed by integer numbers as in `COPY --from=0`,
       starting with 0 for the first stage.  They can also have names
       by using the `FROM image AS stage_name` pattern, and then be
       referenced by names as in `COPY --from=stage_name`.
     + Docker will run the stages in parallel unless there are
       dependencies among them as specified in `COPY
       --from=stage_name`.
     + `docker build` provides the `--target` option to specify the
       stage needed to build, instead of building all stages by
       default.
     + Any existing images can be used as a stage, not only the stages
       defined in the Dockerfile, as in `COPY --from=ubuntu:latest`.
     + Since a stage will result in an intermediate image, so any
       stages can be used by subsequent stages as the base image, such
       as `FROM stage_name`.
     + See also [Multi-stage
       Builds](https://docs.docker.com/build/building/multi-stage/).
   * Mounts can be used in Dockerfiles when building images, such as
     cache mounts `--mount=type=cache,target=xxx` and bind mounts
     `--mount=type=bind,source=xxx,target=yyy`.  See also [Mounts -
     Build with Docker](https://docs.docker.com/build/guide/mounts/).
   * Build arguments defined by the
     [`ARG`](https://docs.docker.com/engine/reference/builder/#arg)
     instruction can be used to parameterize a Dockerfile.
     1. Define a build argument.  For example,

        ```Dockerfile
        ARG VERSION=latest
        ```

     1. Use the argument somewhere in the Dockerfile.  For example,

        ```Dockerfile
        FROM ubuntu:${VERSION}
        ```

     1. Provide a value at build time if it needs to be different from
        the default one.  For example,
        
        ```bash
        docker build --build-arg="VERSION=22.04"
        ```

   * Sometimes, we just want to use Docker container as an isolated
     environment like a virtual machine to compile souce code and get
     the binaries only, instead of a Docker image.  To do that, we
     could:
     1. Use a scratch stage to copy those binaries somewhere, so that
        the image built from the stage contains only those binaries.

        ```Dockerfile
        FROM scratch AS binaries
        COPY --from=previous-stage xxx yyy
        ```

     1. Export only the binaries from the image to somewhere locally
        on the host by using the
        [`--output`](https://docs.docker.com/engine/reference/commandline/image_build/#output)
        option of `docker build`.

        ```bash
        docker build --output=. --target=binaries .
        ```

     See also [Export binaries -- Build with
     Docker](https://docs.docker.com/build/guide/export/).

   </details>

1. Prepare all the other files needed.  For example, `hello.py`:

   ```python
   import os
   print("Hello " + str(os.getenv("NAME", "World")))
   ```

1. Build the image according to the Dockerfile:

   ```console
   $ # Build an image from the Dockerile in current directory and tag it <image-tag-name>
   $ # Usually a tag name is like author/repo:tag
   $ docker build -t <image-tag-name> .
   ```
   
1. Finally, you can run the image by its tag name:

   ```console
   $ docker run <image-tag-name>
   ```

</details>


### Via container ###

<details>
<summary>Click to see more ...</summary>

1. Run a container.  Usually we need a base image to start, such as
   `ubuntu`:

   ```console
   $ # --name gives a name to the container in order to refer to it afterwards
   $ # -it    enables interaction to get into the container
   $ # Usually a image tag name is like author/repo:tag, so when we say:
   $ #     docker run ubuntu
   $ # we refer to ubuntu:latest
   $ docker run --name mlhub -it ubuntu:latest
   ```

1. Setup the environment inside the container.

   ```console
   root@xxx $ pip3 install mlhub
   ```
   
1. Commit the changes in the container into an image.

   ```
   $ # -m    gives a commit message
   $ # -a    gives the author of the image
   $ # mlhub is the container name.
   $ # mlhubber/mlhub:v2.0 is the tag name for the newly created image.
   $ docker commit -m "Added mlhub" -a "mlhubber" mlhub mlhubber/mlhub:v2.0
   ```

</details>


### Reference ###

- [Docker Tutorial: Get Going From Scratch](https://stackify.com/docker-tutorial/)
- [Getting Started with Docker](https://scotch.io/tutorials/getting-started-with-docker)

</details>


## Make a base image ##

<details>
<summary>Click to see more ...</summary>

Usually we make an Docker image from a **parent image** (such as `FROM
ubuntu`).  If we want to make our own parent image which is called
[**base image**](https://docs.docker.com/glossary/#base-image) and has
not parent image, there are 2 ways:
1. Make a minimal OS full image 
1. Use `FROM scratch`


### Make a minimal OS image ###

<details>
<summary>Click to see more ...</summary>

Here we use `debootstrap` to make a minimal Debian image.  **NOTE**:
The system generated in this way still needs extra tuning in order to
function as a full OS system.

```console
$ # Install debootstrap, a tool for making a minimal Debian system
$ sudo apt install debootstrap

$ # Make a bionic system into the directory minisys.
$ # The size of it will be about 307MB
$ sudo debootstrap bionic minisys

$ # Package minisys into a Docker image called 'bionic'
$ # The generated Docker image is around 289MB
$ sudo tar -C minisys -c . | docker import - bionic
```


#### Reference ####

- [Create a base image](https://docs.docker.com/develop/develop-images/baseimages/)
- [moby/contrib/mkimage/debootstrap](https://github.com/moby/moby/blob/master/contrib/mkimage/debootstrap)
- [Installing Debian GNU/Linux from a Unix/Linux System](https://www.debian.org/releases/jessie/amd64/apds03.html.en)

</details>


### Use `FROM scratch` ###

<details>
<summary>Click to see more ...</summary>

Here we use a C++ 'Hello' as an example.  Save the following code as
`hello.sh`.

```bash
#!/bin/bash -

# Generate cpp source code
cat <<EOF > hello.cpp
#include<iostream>
int main() {
  std::cout << "Hello!" << std::endl;
  return 0;
}
EOF

# Compile and generate static executable file
g++ -o hello -static hello.cpp

# Generate Dockerfile
cat <<EOF > Dockerfile
FROM scratch    # Use scratch to make image
ADD hello /     # Put the executable generated previously into the dir / of the iamge
CMD ["/hello"]  # Set the executable as the default command invoked when running the image
EOF

# Make the image according to the Dockerfile above
docker build -t hello .

# Remove intermediate files
rm hello hello.cpp Dockerfile
```

Then run the script `hello.sh`:

```console
$ bash hello.sh 
Sending build context to Docker daemon  2.255MB
Step 1/3 : FROM scratch
 ---> 
Step 2/3 : ADD hello /
 ---> 6844b59b14fb
Step 3/3 : CMD ["/hello"]
 ---> Running in ce8506ded932
Removing intermediate container ce8506ded932
 ---> 362d7a6980b5
Successfully built 362d7a6980b5
Successfully tagged hello:latest
$ docker history hello
IMAGE         CREATED      CREATED BY                         SIZE    COMMENT
362d7a6980b5  2 hours ago  /bin/sh -c #(nop)  CMD ["/hello"]  0B
6844b59b14fb  2 hours ago  /bin/sh -c #(nop) ADD file:968...  2.25MB
```

We can see that the image is built layer by layer.  That is the result
of a Dockerfile directive is a layer in the image, which is a point in
the history of the image, such as `6844b59b14fb` of the above output,
and is an intermediate image as well.  Docker will cache the
intermediate image, thus when you change the order of directives in
the Dockerfile, Docker will re-use them if possible.  Therefore, you'd
better place at the bottom the directives that will change frequently.


#### Reference ####

* [Create a base image](https://docs.docker.com/build/building/base-images/)
* [Optimizing Your Dockerfile](https://medium.com/@esotericmeans/optimizing-your-dockerfile-dc4b7b527756)
* [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
* [Creating a Docker Image from Scratch](https://linuxhint.com/create_docker_image_from_scratch/)
* [Build a Base Image from Scratch](https://docker-k8s-lab.readthedocs.io/en/latest/docker/docker-base-image.html)

</details>

</details>


## Reduce image size ##

<details>
<summary>Click to see more ...</summary>

The basic principle is removing unnecessary files and dependencies.
For example, to make the above C++ 'Hello' image, you don't need GCC
to be included in the image, though GCC is required to build the
source code into the executable.  However, if you put the source code
into the image instead of the executable, you have to install GCC
inside the image to compile the code, and GCC will contribute quite an
amount to the size of the final image.


### Multi-stage builds ###

<details>
<summary>Click to see more ...</summary>

Multi-stage builds can be used to optimize the size of an image:

```dockerfile
# python:3 includes GCC and other libraries to build some Python package
FROM python:3 as python-base
COPY requirements.txt .
RUN pip install -r requirements.txt

# python:3-alpine only includes Python related files
FROM python:3-alpine
# pip will put all downloaded and compiled packages into .cache, thus we 
# only need .cache instead of GCC and other redundant libraries to be 
# included into our final image.
COPY --from=python-base /root/.cache /root/.cache
COPY --from=python-base requirements.txt .
RUN pip install -r requirements.txt && rm -rf /root/.cache
```

Here we use 2-stage builds.  One is used for compiling the packages
needed.  The other is used for installing and packaging the results
into our final image.


#### Reference ####

- [Lighter Python images using multi-stage Dockerfile](https://lekum.org/post/multistage-dockerfile/)
- [Use multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)
- [Advanced multi-stage build patterns](https://medium.com/@tonistiigi/advanced-multi-stage-build-patterns-6f741b852fae)
- [Docker build patterns](https://matthiasnoback.nl/2017/04/docker-build-patterns/)

</details>


### Remove unnecessary files ###

<details>
<summary>Click to see more ...</summary>

Here is some tips:

- Remove cached `.deb` package by `sudo apt-get clean`.
- Remove unused dependencies by `sudo apt-get autoremove`.
- Remove logs in `/var/log` and caches in `/var/cache`.
- Export image:

  ```bash
  docker export <container> | docker import - <image>
  ```

  instead of committing image:

  ```bash
  docker commit -m "Remove unnecessary files to make final image" <container> <image>
  ```

  As mentioned previously, every commit will add a layer into the
  image, so all removed unnecessary files are still in the history of
  the image, which will make the final image bigger instead of
  smaller.
  
- Install package by `apt-get install --no-install-recommends` to
  avoid installing unnecessary recommended packages.
- Remove Conda caches by `conda clean -y -a`
- Make `apt-get update` work together with `apt-get install`, such as
  `apt-get update && apt-get install xxx`, to make a single layer
  inside the image.


#### Reference ####

- [Squeeze disk space on a Debian system](https://ownyourbits.com/2017/02/18/squeeze-disk-space-on-a-debian-system/)
- [Tips to Reduce Docker Image Sizes](https://hackernoon.com/tips-to-reduce-docker-image-sizes-876095da3b34)

</details>


### Use minimization tools ###

- [Skinnywhale helps you make smaller (as in megabytes) Docker containers](https://github.com/djosephsen/skinnywhale)

</details>


## Available base image ##

<details>
<summary>Click to see more ...</summary>

### iron ###

- [Iron](https://hub.docker.com/u/iron/)
- [Uber tiny Docker images for all the things](https://github.com/iron-io/dockers)
- [Microcontainers -- Tiny, Portable Docker Containers](https://blog.iron.io/microcontainers-tiny-portable-containers/)


### phusion ###

- [phusion/baseimage](https://hub.docker.com/r/phusion/baseimage/)
- [A minimal Ubuntu base image modified for Docker-friendliness](https://github.com/phusion/baseimage-docker)
- [Baseimage-docker, fat containers and "treating containers as VMs"](https://blog.phusion.nl/2015/01/20/baseimage-docker-fat-containers-treating-containers-vms/)


### minideb ###

- [bitnami/minideb](https://hub.docker.com/r/bitnami/minideb/)
- [A small image based on Debian designed for use in containers](https://github.com/bitnami/minideb)
- [Minideb: A Minimalist, Debian-Based Docker Image](https://dzone.com/articles/minideb-a-minimalist-debian-based-docker-image)


### miniconda3 ###

It is about 180MB and conda is located in `/opt/conda` inside the
image.

- [continuumio/miniconda3](https://hub.docker.com/r/continuumio/miniconda3/)
- [Conda Environments with Docker](https://medium.com/@chadlagore/conda-environments-with-docker-82cdc9d25754)


### tensorflow ###

It is about 2GB for GPU version, and 500MB for non-GPU.

- [tensorflow/tensorflow](https://hub.docker.com/r/tensorflow/tensorflow/)


### R images ###

- [Opening Reproducible Research](https://o2r.info/)
- [Generating Dockerfiles for reproducible research with R](http://o2r.info/containerit/articles/containerit.html)
- [Rocker](https://github.com/rocker-org/rocker) and [Rocker -- Docker Hub](https://hub.docker.com/u/rocker/)

</details>


## About mount/volume ##

<details>
<summary>Click to see more ...</summary>

Docker image consists of several read-only layers, each of which
corresponds to one of the directives in the Dockerfile.

When we start a container (**container** `cA`) of an image (**image**
`iBase`), Docker will generate a read-write layer (**layer** `lA`) on
top of the topest read-only layer (**layer** `lBase`) of image
`iBase`.  Then all operations afterwards are carried out on the layer
`lA` of container `cA`, and all these changes will disappear after
removing container `cA` without affecting image `iBase` and subsequent
containers generated by running image `iBase`.

The file system of Docker is called **Union File System**.  If you
save the changes on container `cA` into a new image (**iamge** `iA`).
Then image `iA` is consisted of all layers of image `iBase`, plus
layer `lA` on top of layer `lBase`.

If we deleted some files in image `iBase` when creating layer `lA`,
those files won't be seen on the containers generated by running image
`iA`.  That doesn't mean those files are gone.  Instead they are just
hidden in the layer history.  That is the reason why additional layers
added on a image make its size bigger.

For more details about image and container layers, see [Images and
layers](https://docs.docker.com/storage/storagedriver/).

Due to the characteristics of Union FS, we need to persistently save
some files outside of Docker container, that is, on the file system of
the host, which can be solved by using mount/volume.  There are 3 ways
to use mount/volume:
1. Use [**volumes**](https://docs.docker.com/storage/volumes/)
   * There are 3 ways to use volumes:
     1. Map a volume automatically generated and managed by Docker on
        the file sytem of host under `/var/lib/docker/volumes` to a
        directory on the file system of container:

        ```bash
        docker run -v /path/to/a/dir/of/container <image>
        ```

        + This type of mounts is also called **anonymous volumes**.
        + The automatically generated volume directory will be located
          in `/var/lib/docker/volumes` on the host and its mapped
          directory in the container will be
          `/path/to/a/dir/of/container`.
     1. Map an existing volume on the host under
        `/var/lib/docker/volumes` to a directory on the container:

        ```bash
        # Create a volume under /var/lib/docker/volumes and name it <volume>
        docker volume create --name <volume>
        # Check available volumes
        docker volume ls
        # Map the volume to a dir on container
        docker run -v <volume>:/path/to/a/dir/of/container <image>
        ```

        + This type of mounts is called **named volumes**.
     1. Share volumes among containers:

        ```bash
        docker run --volumes-from <container> <image>
        ```

   * Volumes can also be created and mounted by using the recommended
     unified `--mount` option.
   * Unlike bind mounts and regular directories that are dependent on
     and managed by the host filesystem, volumes are special area
     managed by Docker with supports for filesytem behavior consistent
     to that in the container.  It also means, volumes have much
     higher performance than bind mounts from Mac and Windows hosts.
     So in general, you should use volumes wherever possible unless
     the file or directory structure on the host is guaranteed to be
     consistent.
   * If the volume is empty, then files in the directory of the
     container where the volume mounts will copied into the volume; If
     the volume is not empty, then files in the directory of the
     container will be obscured.
   * Volumes are also good to use when you want to store your
     container's data on a remote host or a cloud provider, rather
     than locally.
1. Use [**bind
   mounts**](https://docs.docker.com/storage/bind-mounts/).  This is
   to map an existing directory on the host to a directory on the
   container:

   ```bash
   docker run -v /path/to/a/dir/of/host:/path/to/a/dir/of/container <image>
   ```
   
   * Just as general Linux mount, this will make the files on the host
     obscure/shadow those already existed on the container.  Those
     obscured files are not removed or altered, but are not accessible
     until unmounted.
   * The directory on the host doesn't need to exist, because it will
     be created on demand.
   * It can also be created using the `--mount` option.
1. Use [**tmpfs mounts**](https://docs.docker.com/storage/tmpfs/) on
   Linux only.  This is to map a directory on the host's memory to a
   directory on the container.
   * It is generally used by a container during the lifetime of the
     container, to store non-persistent state or sensitive
     information, for performance or security reasons, instead of
     storing them permannently.


### Examples ###

<details>
<summary>Click to see more ...</summary>

```console
$ docker run -d --rm -it -h container --name ubuntu-volume -v /home/simon ubuntu
f64319429baf
$ docker ps -a
CONTAINER ID  IMAGE   COMMAND      CREATED        STATUS        PORTS  NAMES
f64319429baf  ubuntu  "/bin/bash"  7 seconds ago  Up 6 seconds         ubuntu-volume
$ docker volume ls  # Check available volume generated by Docker
DRIVER              VOLUME NAME
local               b17e59405de1
$ # Use Go Template to check mount information of the volume mapped to 
$ # container 'ubuntu-volume'.
$ # Here jd is a tool for viewing JSON.
$ docker inspect -f "{{json .Mounts}}" ubuntu-volume | jq .
[
  {
    "Type": "volume",
    "Name": "b17e59405de1",
    "Source": "/var/lib/docker/volumes/b17e59405de1/_data",
    "Destination": "/home/simon",
    "Driver": "local",
    "Mode": "",
    "RW": true,
    "Propagation": ""
  }
]
$ sudo ls /var/lib/docker/volumes/b17e59405de1/
_data
$ docker attach ubuntu-volume  # go into container
root@container:/# ls /home/
simon
root@container:/# ls /home/simon/
root@container:/# read escape sequence  # Use Ctrl+Q+P to put container in background
$ docker ps -a
CONTAINER ID  IMAGE   COMMAND      CREATED       STATUS       PORTS  NAMES
f64319429baf  ubuntu  "/bin/bash"  a minute ago  Up a minute         ubuntu-volume
$ # Modify files in the volume on the host
$ sudo bash -c "echo hello > /var/lib/docker/volumes/b17e59405de1/_data/abc"
$ sudo cat /var/lib/docker/volumes/b17e59405de1/_data/abc
hello
$ docker attach ubuntu-volume  # go into container again
root@container:/# cat /home/simon/abc 
hello
root@container:/# cat /home/simon/abc 
hello
root@container:/# ll /home/simon/
total 12
drwxr-xr-x 2 root root 4096 Nov 24 03:58 ./
drwxr-xr-x 1 root root 4096 Nov 24 03:55 ../
-rw-r--r-- 1 root root    6 Nov 24 03:58 abc
root@container:/# echo simon >> /home/simon/abc 
root@container:/# cat /home/simon/abc 
hello
simon
root@container:/# read escape sequence  # Use Ctrl+Q+P to put container in background again
$ docker ps -a
CONTAINER ID  IMAGE   COMMAND      CREATED        STATUS        PORTS  NAMES
f64319429baf  ubuntu  "/bin/bash"  4 minutes ago  Up 4 minutes         ubuntu-volume
$ sudo cat /var/lib/docker/volumes/b17e59405de1/_data/abc  # Check modification 
hello
simon
$ docker volume rm b17e59405de1  # Remove a volume
b17e59405de1
$ docker volume create  # Create a volume
0280dc142b07
$ docker volume ls
DRIVER              VOLUME NAME
local               0280dc142b07
$ docker volume rm 0280dc142b07
0280dc142b07
$ docker volume ls
DRIVER              VOLUME NAME
$ docker volume create --name test  # Create a volume and give it a name
test
$ docker volume ls
DRIVER              VOLUME NAME
local               test
$ docker volume inspect test   # Check the volume info
[
    {
        "CreatedAt": "2018-11-24T15:09:57+08:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/test/_data",
        "Name": "test",
        "Options": {},
        "Scope": "local"
    }
]
$ sudo bash -c 'echo "hello" >> /var/lib/docker/volumes/test/_data/abc'
$ sudo cat /var/lib/docker/volumes/test/_data/abc
hello
$ # New a container, and map a existing volume to a dir inside the container.
$ # Then the files already existed in the volume will shadow those inside the container.
$ docker run -it --rm -h container --name ubuntu-volume -v test:/home/simon ubuntu
e5df1092e96f
root@container:/# cat /home/simon/abc 
hello
root@container:/# exit
$ docker ps -aq
$ docker volume ls -q
test
$ # Map /bin on the host to /bin on the container.
$ # So the binaries in /bin are those on the host
$ docker run -it --rm -h container --name ubuntu-volume -v /bin:/bin ubuntu
root@container:/# ls /bin/
bash           fuser       nisdomainname  stty
brltty         fusermount  ntfs-3g        su
bunzip2        getfacl     ntfs-3g.probe  sync
busybox        grep        ntfscat        systemctl
bzcat          gunzip      ntfscluster    systemd
bzcmp          gzexe       ntfscmp        systemd-ask-password
bzdiff         gzip        ntfsfallocate  systemd-escape
bzegrep        hciconfig   ntfsfix        systemd-hwdb
bzexe          hostname    ntfsinfo       systemd-inhibit
...
root@container:/# exit
$ # See the difference inside /bin on the host above and /bin on the container below
$ docker run -it --rm -h container --name ubuntu-volume ubuntu
root@container:/# ls /bin/
bash          cat            echo      ls             rbash       tempfile      zegrep
bunzip2       chgrp          egrep     lsblk          readlink    touch         zfgrep
bzcat         chmod          false     mkdir          rm          true          zforce
bzcmp         chown          fgrep     mknod          rmdir       umount        zgrep
bzdiff        cp             findmnt   mktemp         run-parts   uname         zless
bzegrep       dash           grep      more           sed         uncompress    zmore
bzexe         date           gunzip    mount          sh          vdir          znew
bzfgrep       dd             gzexe     mountpoint     sh.distrib  wdctl
bzgrep        df             gzip      mv             sleep       which
...
root@container:/# exit
$ docker container prune  # Remove unused container
$ docker volume prune     # Remove unused volume
$ docker ps -aq           # Check containers
$ docker volume ls -q     # Check volumes
$ # New a container and new a volume
$ docker run -itd -h container1 --name container1 -v /home/simon ubuntu
16d720adb189
$ docker ps -aq
16d720adb189
$ docker volume ls -q
3a75da9c2bb17f6be1e7
$ # See, 'destination' only shows in docker inspect container
$ docker volume inspect 3a75da9c2bb17f6be1e7
[
    {
        "CreatedAt": "2018-11-25T10:23:31+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/3a75da9c2bb17f6be1e7/_data",
        "Name": "3a75da9c2bb17f6be1e7",
        "Options": null,
        "Scope": "local"
    }
]
$ # Share container between containers
$ docker run -it -h container2 --name container2 --volumes-from container1 ubuntu
root@container2:/# cd /home/
root@container2:/home# cd simon/
root@container2:/home/simon# ls
root@container2:
$ docker ps -aq
0a4a65c0d8a0
16d720adb189
$ docker attach container1
root@container1:/# echo "hello container1" >> /home/simon/abc
root@container1:/# read escape sequence
$ docker attach container2
root@container2:/home/simon# ls
abc
root@container2:/home/simon# cat abc 
hello container1
root@container2:/home/simon# echo "hello container2" >> abc 
root@container2:/home/simon# read escape sequence
$ docker attach container1
root@container1:/# cat /home/simon/abc 
hello container1
hello container2
root@container1:/# read escape sequence
$ docker stop container1 container2
container1
container2
$ docker ps -aq
0a4a65c0d8a0
16d720adb189
$ # Share volume with a third container
$ docker run -it -h container3 --name container3 --volumes-from container2 ubuntu
root@container3:/# ls /home/simon/abc 
/home/simon/abc
root@container3:/# cat /home/simon/abc 
hello container1
hello container2
root@container3:/# exit
$ # Use volume name to map.  However, the name is different from the previous volume
$ docker run -it -h container4 --name container4 -v 3a75da9c2bb1:/home/data ubuntu
root@container4:/# cd /home/
root@container4:/home# cd data/
root@container4:/home/data# ls 
root@container4:/home/data# exit
$ docker volume ls
DRIVER              VOLUME NAME
local               3a75da9c2bb1
local               3a75da9c2bb17f6be1e7
$ # This is the correct name for the volume to be shared.
$ docker run -it -h container5 --name container5 -v 3a75da9c2bb17f6be1e7:/home/data ubuntu
root@container5:/# ls /home/data/
abc
root@container5:/# cat /home/data/abc 
hello container1
hello container2
root@container5:/# exit
```

</details>

### Other Notes ###

<details>
<summary>Click to see more ...</summary>

If you want files created or changed inside a volume to be valid, you
should put those operations before the `volume` directive:

```dockerfile
from ubuntu

run mkdir /home/data
run echo "hello" >> /home/data/abc

volume /home/data  # volume directive after operations to the files
cmd ["bash"]
```

```dockerfile
from ubuntu

volume /home/data  # volume directive before operations to the files

run echo "hello" >> /home/data/abc
cmd ["bash"]
```

```console
$ # Notice the order between volume directive and the file modification
$ cat Dockerfile
from ubuntu
run mkdir /home/data
run echo "hello" >> /home/data/abc
volume /home/data
cmd ["bash"]
$ docker build -t testvolume .
Sending build context to Docker daemon  2.048kB
Step 1/5 : from ubuntu
 ---> 93fd78260bd1
Step 2/5 : run mkdir /home/data
 ---> Running in 01671d764ca4
Removing intermediate container 01671d764ca4
 ---> dfe4c3ffc537
Step 3/5 : run echo "hello" >> /home/data/abc
 ---> Running in 8834fbe8dbac
Removing intermediate container 8834fbe8dbac
 ---> 60a7d5b878f4
Step 4/5 : volume /home/data
 ---> Running in 0d257a7c9e67
Removing intermediate container 0d257a7c9e67
 ---> 5d0fa2936b9f
Step 5/5 : cmd ["bash"]
 ---> Running in cf66aaef0363
Removing intermediate container cf66aaef0363
 ---> dac633d0f3f5
Successfully built dac633d0f3f5
Successfully tagged testvolume:latest
$ docker ps -aq
$ docker volume ls -q
$ # New a container, mount volume, /home/data/abc does exist
$ docker run -it --rm testvolume
root@b5cd2a2ee79a:/# cd /home/data
root@b5cd2a2ee79a:/home/data# cat abc 
hello
root@b5cd2a2ee79a:/home/data# exit
$ docker volume ls -q
651b43a9bf819ea8ec47
$ mkdir volume
$ echo "simon" >> volume/def
$ docker volume rm 651b43a9bf819ea8ec47
651b43a9bf819ea8ec47
$ docker volume ls -q
$ # If mount a existed dir on the host, /home/data/abc does not exist
$ docker run -it -v ${PWD}/volume:/home/data testvolume
root@bfd2d718e8ca:/# ls /home/data/
def
root@bfd2d718e8ca:/# cat /home/data/def 
simon
root@bfd2d718e8ca:/# exit
$ # Change the order of volume directive and file modification
$ emacs Dockerfile
$ cat Dockerfile
from ubuntu
volume /home/data
run echo "hello" >> /home/data/abc
cmd ["bash"]
$ docker build -t testvolume2 .
Sending build context to Docker daemon  2.048kB
Step 1/4 : from ubuntu
 ---> 93fd78260bd1
Step 2/4 : volume /home/data
 ---> Running in 6c7d8e6b53d8
Removing intermediate container 6c7d8e6b53d8
 ---> 22a4a7cece07
Step 3/4 : run echo "hello" >> /home/data/abc
 ---> Running in 2142946a0638
Removing intermediate container 2142946a0638
 ---> ff893445d3a7
Step 4/4 : cmd ["bash"]
 ---> Running in 187f37a1bbf9
Removing intermediate container 187f37a1bbf9
 ---> 280e29e2a3f6
Successfully built 280e29e2a3f6
Successfully tagged testvolume2:latest
$ # See the modification is invalid
$ docker run -it testvolume2
root@3631906e5e3b:/# ls /home/data/
root@3631906e5e3b:/# cd /home/data/
root@3631906e5e3b:/home/data# ls
```

</details>


### Reference ###

- [Understanding Volumes in Docker](https://container-solutions.com/understanding-volumes-docker/)

</details>


## Compose ##

<details>
<summary>Click to see more ...</summary>

[Docker Compose](https://docs.docker.com/compose/) is a tool for
defining and running multi-container Docker applications.  In other
words, for a multi-container application, without Compose, you have to
manually build Docker images and set up those multiple containers.
But with Compose, you define those steps in a YAML configuration file,
then create and start all the services from configuration file with a
single command `docker-compose up`.  See [Get started with Docker
Compose](https://docs.docker.com/compose/gettingstarted/).

</details>


## Be careful ##

<details>
<summary>Click to see more ...</summary>

- Do not `apt-get upgrade` inside an image if you want your work
  reproducible
- Specify the tag version of the parent image, such as `FROM
  ubuntu:18.04` instead of `FROM ubuntu`


### Reference ###

- [9 Common Dockerfile Mistakes](https://runnable.com/blog/9-common-dockerfile-mistakes)

</details>


## Misc ##

[Apptainer](https://apptainer.org/) (formerly
[Singularity](https://singularity.lbl.gov/)) is a container platform
similar to Docker but for HPC.  See
* [Apptainer Documentation](https://apptainer.org/documentation/)
* [Sigularity User Guide](https://www.sylabs.io/guides/3.0/user-guide/)


## Reference ##

* [Docker Docs](https://docs.docker.com)
  + [Gettting started guide](https://docs.docker.com/get-started/)
  + [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
