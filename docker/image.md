# Docker Image #

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
   # Make sure to 'apt-get update' before 'apt-get install'.
   # See https://docs.docker.com/build/building/best-practices/#apt-get
   # Use 'apt-get' instead of 'apt', otherwise, you will get a warning:
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
     + [`docker image
       build`](https://docs.docker.com/engine/reference/commandline/image_build/)
       (`docker build`) provides the `--target` option to specify the
       stage needed to build, instead of building all stages by
       default.  This is useful especially when you build the testing
       stage and the production stage for testing and production,
       respectively.
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

   * Environment variables defined by the
     [`ENV`](https://docs.docker.com/reference/dockerfile/#env)
     instruction are used to set persistent environment variables.
     + Both `ARG`and `ENV` can be used to set variables used by
       subsequent commands, but variables set `ARG` are only available
       at build time, and persist in the image metadata and in the
       image history, while `ENV` variables are available at both
       build time and runtime, and persist in containers.  So both
       build arguments and environment variables are not suitable for
       [holding
       secrets](https://docs.docker.com/build/building/secrets/).
     + Note that, unsetting `ENV` variables by `RUN unset XXX` only
       take effect on the shell process this `RUN` starts, so
       subsequent `RUN`s will still see the `ENV` variables.
     + More about the differences between environment variables and
       build arguments and how to use them in difference scopes can be
       found at [Build
       variables](https://docs.docker.com/build/building/variables/).
     + `ENV` variables are treated differently when they are used with
       `RUN`, `CMD` and `ENTRYPOINT`, compared to being used with
       other instructions like `ADD`, `FROM`, `COPY`, etc.  The `ENV`
       variable substituion is handled by the command shell when used
       with `RUN`, and the image builder when used with `FROM`.
   * `RUN`, `ENTRYPOINT` and `CMD` accept [2 forms of
     arguments](https://docs.docker.com/reference/dockerfile/#shell-and-exec-form).
     + exect form
       - `INSTRUCTION ["executable", "param1", "param2"]`
     + shell form
       - `INSTRUCTION command param1 param2`
       - In shell form, a default shell is used to invoke the
         `command`.  For example, `RUN echo hello` is equivalent to
         `RUN ["/bin/sh", "-c", "echo hello"]`.
       - The default shell to use can be changed by [the `SHELL`
         instruction](https://docs.docker.com/reference/dockerfile/#shell).
   * [`CMD`](https://docs.docker.com/reference/dockerfile/#cmd) and
     [`ENTRYPOINT`](https://docs.docker.com/reference/dockerfile/#entrypoint)
     are used to set the command to be executed when running a
     container from an image.
     + [`docker
       inspect`](https://docs.docker.com/reference/cli/docker/inspect/)
       can be used to view detailed information of docker objects,
       such as the `CMD` and `ENTRYPOINT` of an image.
       - [The `CMD` of the `ubuntu:24.04`
         image](https://git.launchpad.net/cloud-images/+oci/ubuntu-base/tree/Dockerfile?h=noble-24.04)
         is `["/bin/bash"]`.
       - The `ENTRYPOINT` of the `bash` image is a script
         [`docker-entrypoint.sh`](https://github.com/tianon/docker-bash/blob/master/docker-entrypoint.sh).
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
   
1. Commit the changes in the container into an image by [`docker
   container
   commit`](https://docs.docker.com/reference/cli/docker/container/commit/)
   (`docker commit`).

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
