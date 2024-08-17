# Running Containers #

All container-related commands are collected as subcommands of `docker
container`.

See also [Running
containers](https://docs.docker.com/engine/reference/run/).

* [`docker container
  ls`](https://docs.docker.com/engine/reference/commandline/container_ls/)
  (`docker ps`) -- List avaiable containers.

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

* [`docker container
  run`](https://docs.docker.com/engine/reference/commandline/container_run/)
  (`docker run`) -- Create and run a contaienr from an Docker image.
  + Instead of re-using previous container of the same image, a new
    container will be created every time you run this command, thus
    every container have a unique ID and name which can be referred to
    afterwards.
  + Instead of randomly assigned, A user-specified name can be given
    to the container via the `--name` option for later reference.

    ```console
    $ docker run --name mytest hello-world
    $ docker ps -a
    CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                       NAMES
    43218a4b9223   hello-world   "/hello"   6 seconds ago    Exited (0) 6 seconds ago     mytest
    a318b8ca5283   ubuntu        "bash"     35 minutes ago   Up 34 minutes                trusting_knuth
    $ docker rm mytest
    mytest
    $ docker ps -a
    CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS            NAMES
    a318b8ca5283   ubuntu    "bash"    35 minutes ago   Up 35 minutes     trusting_knuth
    ```

  + The container runs in the foreground by default until the process
    finishes.
    - Containers of some images such as the `bash` image will exit
      immediately when running.
      
      ```console
      $ docker ps -a
      CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS     NAMES
      $ docker run --name mytest bash
      $ docker ps -a
      CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                       NAMES
      f6f667f1fbe0   bash      "docker-entrypoint.s…"   3 seconds ago   Exited (0) 2 seconds ag      mytest
      ```
      
      * To prevent the container exiting, we can use the
        `-i/--interactive` option to keep STDIN open, and add together
        the `-t/--tty` option to attach a pseudo-TTY (terminal), so
        that the input and output feature (such as echo-off) of TTY
        devices can be used, thus we create an interactive terminal
        session for the container.  See also [Keep STDIN
        open](https://docs.docker.com/engine/reference/commandline/container_run/#interactive)
        and [Allocate a
        pseudo-TTY](https://docs.docker.com/engine/reference/commandline/container_run/#tty).
      * To escape the interactive terminal session but leave the
        container run in the backgroud without exiting, we can type
        `Ctrl + P Ctrl+ Q`.  If we use `Ctrl + D` when in the terminal
        session of the container, the container will exit intead of
        keeping running in the background.
      * To re-attach to the interactive termianl session running in
        the backgroud, we can use [`docker container
        attach`](https://docs.docker.com/engine/reference/commandline/container_attach/)
        (`docker attach`).
      * To make the container running in the background without
        exiting, we can use the `-d/--detach` option toghter with
        `-it` to make it run in detached mode.
      
      ```console
      $ docker run --name mytest -it bash
      bash-5.2# pwd  # Now we are in the interactive terminal session of the contaienr
      /
      bash-5.2#  # Type Ctrl + P Ctrl + Q to escape
      $ docker ps
      CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS            NAMES
      1644696b8cac   bash      "docker-entrypoint.s…"   14 minutes ago   Up 14 minute      mytest
      $ docker attach mytest
      bash-5.2# pwd
      /
      bash-5.2#  # Type Ctrl + P Ctrl + Q to escape
      $ docker ps
      CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS            NAMES
      1644696b8cac   bash      "docker-entrypoint.s…"   25 minutes ago   Up 25 minute      mytest
      $ docker rm -f mytest
      mytest
      $ docker ps
      CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
      $ docker run --name mytest -it -d bash
      dc45b105b4192c04b30476cdf5eaa82cb3b1c87d954fcc4a8d24b07790d1b6d3
      $ docker ps
      CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS           NAMES
      dc45b105b419   bash      "docker-entrypoint.s…"   2 seconds ago   Up 2 second      mytest
      $ docker attach mytest
      bash-5.2# pwd
      /
      bash-5.2# 
      ```

  + The `-a/--attach` option can be used to specify which of STDIN,
    STDOUT and STDERR of the contaienr needs to be connected to the
    host.
  
    ```
    # Attach STDIN and STDOUT only
    docker run -a stdin -a stdout <IMAGE-TAG>
    ```

    - It is usually used when containers are piped because the attach
      is transient.  To keep STDIN attached and open all the time, use
      the `-i/--interactive` option instead.
    - See also [Understanding docker run --attach
      option](https://forums.docker.com/t/understanding-docker-run-attach-option/134337/4)
      and [Attach to
      STDIN/STDOUT/STDERR](https://docs.docker.com/engine/reference/commandline/container_run/#attach)
* [`docker container
  inspect`](https://docs.docker.com/engine/reference/commandline/container_inspect/)
  -- View the details of a container.
* [`docker container
  logs`](https://docs.docker.com/engine/reference/commandline/container_logs/)
  (`docker logs`) -- View the container logs.
* To mount a directory on the host to a directory in the container:

  ```bash
  # There is another new recommended option `--mount` for `-v`
  docker run -v <HOST-PATH>:<CONTAINER-PATH> <IMAGE-TAG>
  ```

* [`docker container
  exec`](https://docs.docker.com/engine/reference/commandline/container_exec/)
  (`docker exec`) -- Run a command in a running container.

  ```bash
  docker exec <CONTAINER-ID> <COMMAND>
  ```
  
  + Use the `-it` option, we can run the command interatcitvely.

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
