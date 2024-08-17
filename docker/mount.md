# About Mounts #

Docker image consists of several **read-only layers**, each of which
corresponds to one of the directives in the Dockerfile.  When we start
a container `cA` from an image `iBase`, Docker will generate a
**read-write layer** `lA` on top of the topest read-only layer `lBase`
of image `iBase`.
* Then all operations afterwards are carried out on the layer `lA` of
  container `cA`, and all these changes will disappear after removing
  container `cA`, without affecting image `iBase` and subsequent
  containers generated by running image `iBase`.
* The file system of Docker is called **Union File System**.  If you
  save the changes on container `cA` into a new image `iA`.  Then
  image `iA` is consisted of all layers of image `iBase`, plus layer
  `lA` on top of layer `lBase`.
* If we deleted some files in image `iBase` from layer `lA` before
  saving it into image `iA`, those files won't be seen on the
  containers generated by running image `iA`.  That doesn't mean those
  files are gone.  Instead they are just hidden in the layer history.
  That is the reason why additional layers added on a image make its
  size bigger.
* For more details about image and container layers, see [Images and
  layers](https://docs.docker.com/storage/storagedriver/).

Due to the characteristics of Union FS, we need to persistently save
some files outside of Docker container, that is, on the file system of
the host.  Docker provides 3 ways to **mount** a directory onto a
container:
* Volumes
* Bind mounts
* tmpfs mounts

## [Volumes](https://docs.docker.com/storage/volumes/) ##

* Volumes are Docker-managed directories on your local host, which can
  be used in 3 ways.
  1. **Anonymous volumes**

     ```bash
     docker run -v /path/to/a/dir/of/container <image>
     ```

     The command map a anonymous volume automatically generated and
     managed by Docker on the file sytem of host under
     `/var/lib/docker/volumes` to a directory
     `/path/to/a/dir/of/container` on the file system of container.
  1. **Named volumes**

     ```bash
     # Create a volume under /var/lib/docker/volumes and name it <volume>
     docker volume create --name <volume>

     # Check available volumes
     docker volume ls

     # Map the volume to a dir on container
     docker run -v <volume>:/path/to/a/dir/of/container <image>
     ```

     The command first create a named volume under
     `/var/lib/docker/volumes`, and then map the volume to a directory
     `/path/to/a/dir/of/container` on the container.
  1. Share volumes among containers:

     ```bash
     docker run --volumes-from <container> <image>
     ```

* Volumes can also be created and mounted by using the recommended
  unified `--mount` option.
* Unlike bind mounts and regular directories that are dependent on and
  managed by the host filesystem, volumes are special area managed by
  Docker with supports for filesytem behavior consistent to that in
  the container.  It also means, volumes have much higher performance
  than bind mounts from Mac and Windows hosts.  So in general, you
  should use volumes wherever possible unless the file or directory
  structure on the host is guaranteed to be consistent.
* If the volume is empty, then files in the directory of the container
  where the volume mounts will copied into the volume; If the volume
  is not empty, then files in the directory of the container will be
  obscured.
* Volumes are also good to use when you want to store your container's
  data on a remote host or a cloud provider, rather than locally.


## [Bind mounts](https://docs.docker.com/storage/bind-mounts/) ##

Mapping an existing directory on the host to a directory on the
container is called **bind mount**.

```bash
docker run -v /path/to/a/dir/of/host:/path/to/a/dir/of/container <image>
```

* Just as general Linux mount, this will make the files on the host
  obscure/shadow those already existed on the container.  Those
  obscured files are not removed or altered, but are not accessible
  until unmounted.
* The directory on the host doesn't need to exist, because it will be
  created on demand.
* It can also be done by using the recommended unified `--mount`
  option.


## [tmpfs mounts](https://docs.docker.com/storage/tmpfs/) ##

Mapping a directory on the host's memory to a directory on the
container is called **tmpfs mounts**.
* It is supported on Linux only.
* It is generally used by a container, during the lifetime of the
  container, to store non-persistent state or sensitive information,
  for performance or security reasons, instead of storing them
  permannently.


## Examples ##

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

## Other Notes ##

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


## Reference ##

- [Understanding Volumes in Docker](https://container-solutions.com/understanding-volumes-docker/)
