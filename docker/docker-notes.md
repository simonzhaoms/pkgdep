# Docker notes #

- **Docker** is a platform for developers and sysadmins to develop,
  deploy, and run applications with containers.
- An **image** is an executable package that includes everything
  needed to run an application--the code, a runtime, libraries,
  environment variables, and configuration files.
- A **container** is a runtime instance of an image--what the image
  becomes in memory when executed (that is, an image with state, or a
  user process).  It is a lightweight virtual machine.

Docker container can be used in the scenarios:

- You want to test your code.  Take Python application as example.
  You have to make sure you have installed the right Python version,
  all dependent Python packages, as well as system dependencies.  With
  Docker, all these can be packed into a single Docker image, and run
  on several containers without interfere with each other.

[Singularity](https://singularity.lbl.gov/) is a container similar to
Docker but for HPC.  See [User
Guide](https://www.sylabs.io/guides/3.0/user-guide/).


## Contents ##

* [Installation on Ubuntu](#installation-on-ubuntu)
* [Docker commands](#docker-commands)
* [Make a Docker image](#make-a-docker-image)
  + [Via `Dockerfile`](#via-dockerfile)
  + [Via container](#via-container)
* [Make a base image](#make-a-base-image)
  + [Make a minimal OS image](#make-a-minimal-os-image)
  + [Use `FROM scratch`](#use-from-scratch)
* [Reduce image size](#reduce-image-size)
  + [Multi-stage builds](#Multi-stage-builds)
  + [Remove unnecessary files](#remove-unnecessary-files)
  + [Use minimization tools](#use-minimization-tools)
* [Available base image](#available-base-image)
  + [iron](#iron)
  + [phusion](#phusion)
  + [minideb](#minideb)
  + [miniconda3](#miniconda3)
  + [tensorflow](#tensorflow)
  + [R images](#r-images)
* [Be careful](#be-careful)
* [Reference](#reference)


## Installation on Ubuntu ##

There are 2 versions: Community Edition (CE) and Enterprise Edition
(EE).  See [Get Docker CE for
Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/) for
more detailed installation of Docker CE.  **NOTE** There is no apt
source for Ubuntu 18.10.

```bash
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce  # install the latest version

# add current user to group docker in order not to sudo when using docker command
sudo usermod -aG docker $USER

# configure to start docker on startup
sudo systemctl enable docker  # disable by 'sudo systemctl disable docker'

# Test if docker is installed properly
docker run hello-world
```


## Docker commands ##

```console
$ docker            # print out docker usage, equivalent to: docker --help
$ docker --version  # print out docker version
$ docker version    # more detailed version

$ # run image <xxx>, equivalent to:
$ #     docker container run xxx
$ # if you want to remove the container automatically after finishing running:
$ #     docker run --rm xxx

$ # Note: A new container will be created every time you run this command
$ #       instead of re-using previous container of the same image, thus every 
$ #       container have a unique ID and name which can be refered to afterwards.
$ docker run xxx

$ # list available images downloaded
$ docker images ls
REPOSITORY   TAG     IMAGE ID      CREATED      SIZE
hello-world  latest  4ab4c602aa5e  5 weeks ago  1.84kB

$ # list all containers, including those finishing running, equivalent to:
$ #     docker ps -a
$ docker container ls -a
CONTAINER ID  IMAGE        COMMAND   CREATED       STATUS                   PORTS  NAMES
caf880f16684  hello-world  "/hello"  16 hours ago  Exited (0) 16 hours ago         eager_cori

$ # list only container ID
$ docker container ls -aq
caf880f16684

$ # remove all container, equivalent to
$ #     docker container rm $(docker container ls -aq)
$ # if you want to remove only those container stopping running:
$ #     docker container rm $(docker container ls -aq -f status=exited)
$ # or
$ #     docker container prune
$ docker rm $(docker container ls -aq)

$ # print out the log of container caf880f16684
$ # add -f to follow the log tail
$ docker logs caf880f16684

$ # rerun the container 4c40d0c1c838 which stoped beforehand, and attach it, 
$ # so you can interact with it, otherwise, without --attach, it will run in background.
$ # it is equivalent to:
$ #     docker container start --attach 4c40d0c1c838
$ docker start --attach 4c40d0c1c838

$ # run the application 'bash' in the image 'ubuntu', where -it denotes interactive.
$ # it is equivalent to:
$ #     docker container run -it ubuntu bash
$ # if you want it run in background, use option -d, which means dettach:
$ #     docker run -it -d ubuntu bash
$ # usually the hostname of container is its ID, if you want the container have a 
$ # different hostname, use option -h:
$ #     docker run -h mycontainer -it ubuntu bash
$ docker run -it ubuntu bash
root@5856302872b0 $ # here we are in the container, you can press Ctrl + Q + P to 
root@5856302872b0 $ # leave it run in background.
$ # or open another terminal, you will see the container is in running.
$ # to interact with a container run in background, you can attach it:
$ #     docker attach naughty_ptolemy
$ docker ps -a
CONTAINER ID  IMAGE        COMMAND   CREATED         STATUS                     PORTS  NAMES
5856302872b0  ubuntu       "bash"    6 minutes ago   Up 6 minutes                      naughty_ptolemy
4c40d0c1c838  hello-world  "/hello"  17 minutes ago  Exited (0) 13 minutes ago         thirsty_minsky

$ # stop a container, equivalent to:
$ #     docker container stop naughty_ptolemy
$ docker stop naughty_ptolemy
$ docker ps -a
CONTAINER ID  IMAGE        COMMAND   CREATED      STATUS                    PORTS  NAMES
5856302872b0  ubuntu       "bash"    2 hours ago  Exited (0) 4 seconds ago         naughty_ptolemy
4c40d0c1c838  hello-world  "/hello"  2 hours ago  Exited (0) 2 hours ago           thirsty_minsky

$ # remove a container by its name. if the container is still running:
$ #     docker rm -f thirsty_minsky
$ docker rm thirsty_minsky
thirsty_minsky
$ docker ps -a
CONTAINER ID  IMAGE        COMMAND   CREATED      STATUS                    PORTS  NAMES
5856302872b0  ubuntu       "bash"    2 hours ago  Exited (0) 4 minutes ago         naughty_ptolemy

$ # run a container of image hello-world and give it a name 'mytest', equivalent to:
$ #     docker container run --name mytest hello-world
$ docker run --name mytest hello-world
$ docker ps -a
CONTAINER ID  IMAGE        COMMAND   CREATED        STATUS                    PORTS  NAMES
f3912fdb365b  hello-world  "/hello"  5 seconds ago  Exited (0) 3 seconds ago         mytest
5856302872b0  ubuntu       "bash"    2 hours ago    Exited (0) 6 minutes ago         naughty_ptolemy
```


### Reference ###

- [Get Started with Docker](https://docs.docker.com/get-started/)
- [Docker Tutorial: Get Going From Scratch](https://stackify.com/docker-tutorial/)
- [Docker Tutorial: Play With Containers](https://dzone.com/articles/docker-tutorial-play-with-containers-simple-exampl)
- [Docker Tutorial -- Getting Started with Python, Redis, and Nginx](https://hackernoon.com/docker-tutorial-getting-started-with-python-redis-and-nginx-81a9d740d091)
- [docker-curriculum.com](https://docker-curriculum.com/)
- [R Docker tutorial](http://ropenscilabs.github.io/r-docker-tutorial/)


## Make a Docker image ##

There are 2 ways:
- Write a `Dockerfile`.  Describe all the commands used to create the
  image in the `Dockerfile`.
- Save a container into an image.  Setup the environment in a
  container, then commit it into an image.


### Via `Dockerfile` ###

1. Write `Dockerfile`, mixed with commands and `Dockerfile`
   directives.  For example:

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

1. Prepare all the other files needed.  For example, `hello.py`:

   ```python
   import os
   print("Hello " + str(os.getenv("NAME", "World")))
   ```

1. Build the image according to the `Dockerfile`:

   ```console
   $ # Build an image from the Dockerile in current directory and tag it <image-tag-name>
   $ # Usually a tag name is like author/repo:tag
   $ docker build -t <image-tag-name> .
   ```
   
1. Finally, you can run the image by its tag name:

   ```console
   $ docker run <image-tag-name>
   ```


### Via container ###

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


### Reference ###

- [Docker Tutorial: Get Going From Scratch](https://stackify.com/docker-tutorial/)
- [Getting Started with Docker](https://scotch.io/tutorials/getting-started-with-docker)


## Make a base image ##

Usually we make an Docker image from a **parent image** (such as `FROM
ubuntu`).  If we want to make our own parent image which is called
**base image**, there are 2 ways:
- Make a minimal OS full image 
- Use `FROM scratch`


### Make a minimal OS image ###

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


### Use `FROM scratch` ###

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
of a `Dockerfile` directive is a layer in the image, which is a point
in the history of the image, such as `6844b59b14fb` of the above
output, and is an intermediate image as well.  Docker will cache the
intermediate image, thus when you change the order of directives in
the `Dockerfile`, Docker will re-use them if possible.  Therefore,
you'd better place at the bottom the directives that will change
frequently.


#### Reference ####

- [Optimizing Your `Dockerfile`](https://medium.com/@esotericmeans/optimizing-your-dockerfile-dc4b7b527756)
- [Best practices for writing `Dockerfile`s](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Create a base image](https://docs.docker.com/develop/develop-images/baseimages/)
- [Creating a Docker Image from Scratch](https://linuxhint.com/create_docker_image_from_scratch/)
- [Build a Base Image from Scratch](https://docker-k8s-lab.readthedocs.io/en/latest/docker/docker-base-image.html)


## Reduce image size ##

The basic principle is removing unnecessary files and dependencies.
For example, to make the above C++ 'Hello' image, you don't need GCC
to be included in the image, though GCC is required to build the
source code into the executable.  However, if you put the source code
into the image instead of the executable, you have to install GCC
inside the image to compile the code, and GCC will contribute quite an
amount to the size of the final image.


### Multi-stage builds ###

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

- [Lighter Python images using multi-stage `Dockerfile`](https://lekum.org/post/multistage-dockerfile/)
- [Use multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)
- [Advanced multi-stage build patterns](https://medium.com/@tonistiigi/advanced-multi-stage-build-patterns-6f741b852fae)
- [Docker build patterns](https://matthiasnoback.nl/2017/04/docker-build-patterns/)


### Remove unnecessary files ###

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


### Use minimization tools ###

- [Skinnywhale helps you make smaller (as in megabytes) Docker containers](https://github.com/djosephsen/skinnywhale)


## Available base image ##


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


## Be careful ##

- Do not `apt-get upgrade` inside an image if you want your work
  reproducible
- Specify the tag version of the parent image, such as `FROM
  ubuntu:18.04` instead of `FROM ubuntu`


### Reference ###

- [9 Common Dockerfile Mistakes](https://runnable.com/blog/9-common-dockerfile-mistakes)


## Reference ##

- [Docker Docs](https://docs.docker.com)

