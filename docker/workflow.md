# Workflow #

The typical workflow using Docker for development is:
1. Create a **Dockerfile**.
   * A Dockerfile is used to define the environment containerizing
     your application.
   * More details on how to write a Dockerfile can be found at [make a
     Docker image](#make-a-docker-image).
1. Build a **Docker image**.

   ```bash
   docker build -t <IMAGE-TAG> <DOCKERFILE-DIRECTORY>
   ```

   * The command looks for a file called `Dockerfile` under
     `<DOCKERFILE-DIRECTORY>`, build a Docker image from the
     Dockerfile, and git the image a tag `<IMAGE-TAG>`.
   * We can also directly specify the Dockerfile by the `-f
     <DOCKERFILE>` option.
   * [`docker image
     build`](https://docs.docker.com/engine/reference/commandline/image_build/)
     (`docker build`) -- Build a Docker image.
   * [`docker image
     ls`](https://docs.docker.com/engine/reference/commandline/image_ls/)
     (`docker images`) -- List available Docker images.
       
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

   * [`docker image
     history`](https://docs.docker.com/engine/reference/commandline/image_history/)
     (`docker history`) -- View the layers in a Docker image.
1. Run a container

   ```bash
   docker run <IMAGE-TAG>
   ```

   * The command run a Docker container from the image `<IMAGE-TAG>`
     to test your application.
   * [`docker container
     run`](https://docs.docker.com/engine/reference/commandline/container_run/)
     (`docker run`) -- Run a Docker container.
   * More details can be found at [Running
     containers](#running-containers).
1. Revise the Dockerfile.
1. Stop and remove the container.

   ```bash
   docker stop <CONTAINER-ID>
   docker rm <CONTAINER-ID>
   ```

   * The commands stop and remove the container `<CONTAINER-ID>`.  The
     2 commands above can be completed by the single command `docker
     rm -f <CONTAINER-ID>`.
   * By default, a container's file system persists after the
     container exits so that we can inspect the container for
     debugging.  That's why we use `docker rm` to delete the container
     to save spaces.  To let Docker automatically clean up the
     container file system when it exits, we can use [`docker run
     --rm`](https://docs.docker.com/engine/reference/commandline/container_run/#rm)
     when running a container.
   * [`docker container
     stop`](https://docs.docker.com/engine/reference/commandline/container_stop/)
     (`docker stop`) -- Stop a running container.
   * [`docker container
     rm`](https://docs.docker.com/engine/reference/commandline/container_rm/)
     (`docker rm`) -- Remove a container.
   * [`docker container
     start`](https://docs.docker.com/engine/reference/commandline/container_start/)
     (`docker start`)
     + Re-run a stopped container.
     + We can attach to interact with the container by using the
       `-a/--attach` option.
   * Useful commands for removal:
   
     ```bash
     # Remove all containers
     docker rm $(docker ps -aq)
     
     # Remove only those containers stopping running
     docker rm $(docker ps -aq -f "status=exited")
     docker container prune
     ```

1. Run a new container.
   * Create and run a new container from the new image to test the
     update.
1. Share the image.
   * [`docker image
     push`](https://docs.docker.com/engine/reference/commandline/image_push/)
     (`docker push`) -- Push the image to a Docker registry such as
     Docker Hub.
