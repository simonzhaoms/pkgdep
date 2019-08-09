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
* [About volume](#about-volume)
* [Be careful](#be-careful)


## Installation on Ubuntu ##

There are 2 versions: Community Edition (CE) and Enterprise Edition
(EE).  See [Get Docker CE for
Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/) for
more detailed installation of Docker CE.

```bash
# Update APT package index in case of new fresh install
sudo apt-get update
# Install necessary dependencies
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# Set APT repo source
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# Install Docker community edition
sudo apt-get install -y docker-ce docker-ce-cli containerd.io  # install the latest version

# add current user to group docker in order not to use sudo everytime when using docker command
sudo usermod -aG docker $USER

# Test if docker is installed properly.
# If not adding to docker group, prefix 'sudo' to the command below.
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


## About volume ##

Docker image consists of several read-only layers, which are
corresponding to the directives in the `Dockerfile`.

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
hidden in the history.  That is the reason why additional layers added
on a image make its size bigger.

Due to the characteristics of Union FS, we need to persistently save
some files outside of Docker container to the file system of the host,
which can be solved by using volume.  The usage of Volume are:

- Map a directory automatically generated by Docker on the file sytem
  of host under `/var/lib/docker/volumes` to a directory on the file
  system of container:

  ```bash
  docker run -v /path/to/a/dir/of/container <image>
  ```

  The directory `H` will be located in `/var/lib/docker/volumes`.

- Map an existing volume on the host under `/var/lib/docker/volumes`
  to a directory on the container:

  ```bash
  # Create a volume under /var/lib/docker/volumes and name it <volume>
  docker volume create --name <volume>
  # Check available volumes
  docker volume ls
  # Map the volume to a dir on container
  docker run -v <volume>:/path/to/a/dir/of/container <image>
  ```

- Map an existing directory on the host to a directory on the
  container:

  ```bash
  docker run -v /path/to/a/dir/of/host:/path/to/a/dir/of/container <image>
  ```
  
  Just as mount, this will make the files on the host shadow those
  already existed on the container.

- Share volumes among containers:

  ```bash
  docker run --volumes-from <container> <image>
  ```


### Examples ###

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


### Be careful ###

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

### Reference ###

- [Understanding Volumes in Docker](https://container-solutions.com/understanding-volumes-docker/)


## Be careful ##

- Do not `apt-get upgrade` inside an image if you want your work
  reproducible
- Specify the tag version of the parent image, such as `FROM
  ubuntu:18.04` instead of `FROM ubuntu`


### Reference ###

- [9 Common Dockerfile Mistakes](https://runnable.com/blog/9-common-dockerfile-mistakes)


## Reference ##

- [Docker Docs](https://docs.docker.com)

