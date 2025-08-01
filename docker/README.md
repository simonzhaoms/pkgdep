# Docker #

* [Installation](./install.md)
* [Workflow Using Docker for Development](./workflow.md)
* [Running Containers](./container.md)
* [Image Creation](./image.md)
* [About Mounts](./mount.md)


## Reference ##

* [Docker Docs](https://docs.docker.com)
  + [Docker CLI reference](https://docs.docker.com/reference/cli/docker/)
    - Reference for the docker commands such as `docker build`,
      `docker run`.
  + [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)


## Glossary ##

See
* [Docker Engine overview](https://docs.docker.com/engine/)
* [Docker Glossary](https://docs.docker.com/glossary/)

* **Docker**
  + is a platform that provides the tools to package and run an
    application in an environment called a **container** that is
    isolated from the host like a hypervisor-based virtual machine
    (such as VirtualBox) but is lightweight.  See also 
    - [Use containers to Build, Share and Run your applications](https://www.docker.com/resources/what-container/)
    - [Docker -- Glossary](https://docs.docker.com/glossary/#docker)
  + uses a client-server arthitecutre:
    - **The Docker daemon** (also called **Docker Engine** or **Docker
      CE**)
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
    built from a Dockerfile.  In other words, an image is a collection
    of files plus container runtime execution parameters.
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
  + is a place storing Docker images, so that Docker images can be
    shared or distributed via [image
    registries](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-registry/).
  + **Docker Hub** (https://hub.docker.com) is a public registry that
    Docker looks for images by default.
    - In the registry, related images are collected as repositores.
  + In addition, there are many other registries, including
    - [Amazon Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/)
    - [Azure Container Registry (ACR)](https://azure.microsoft.com/en-in/products/container-registry)
    - [Google Container Registry (GCR)](https://cloud.google.com/artifact-registry)
  + Many organizations also set up their own private registries.
  + Docker provides `registry-mirrors` [the Docker daemon
    configuration
    file](https://docs.docker.com/reference/cli/dockerd/#daemon-configuration-file)
    to specify the registry.
  + In some countries, such as China, certain public registries are
    not accessible, so developers [set up their own Docker Hub
    mirrors](https://docs.docker.com/docker-hub/image-library/mirror/).
    - One of the mirrors are [DaoCloud public image
      mirror](https://github.com/DaoCloud/public-image-mirror/tree/main).
