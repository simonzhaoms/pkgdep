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


## Installation on Ubuntu ##

There are 2 versions: Community Edition (CE) and Enterprise Edition
(EE).  See [Get Docker CE for
Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/) for
more detailed installation of Docker CE.

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

###  ###



## Reference ##

- [Docker Docs](https://docs.docker.com)

