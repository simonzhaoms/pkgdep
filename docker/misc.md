# Misc #

## Compose ##

[Docker Compose](https://docs.docker.com/compose/) is a tool for
defining and running multi-container Docker applications.  In other
words, for a multi-container application, without Compose, you have to
manually build Docker images and set up those multiple containers.
But with Compose, you define those steps in a YAML configuration file,
then create and start all the services from configuration file with a
single command `docker-compose up`.  See [Get started with Docker
Compose](https://docs.docker.com/compose/gettingstarted/).


## Be careful ##

- Do not `apt-get upgrade` inside an image if you want your work
  reproducible
- Specify the tag version of the parent image, such as `FROM
  ubuntu:18.04` instead of `FROM ubuntu`


### Reference ###

- [9 Common Dockerfile Mistakes](https://runnable.com/blog/9-common-dockerfile-mistakes)


## Misc ##

[Apptainer](https://apptainer.org/) (formerly
[Singularity](https://singularity.lbl.gov/)) is a container platform
similar to Docker but for HPC.  See
* [Apptainer Documentation](https://apptainer.org/documentation/)
* [Sigularity User Guide](https://www.sylabs.io/guides/3.0/user-guide/)
