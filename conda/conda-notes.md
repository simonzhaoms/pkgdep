# Anaconda #

[Anaconda](https://docs.anaconda.com/anaconda/) is:

- A package manager.  It can search, manage and install Python
  packages.  It add supports to R package management as well (See
  [Using R language with
  Anaconda](https://docs.anaconda.com/anaconda/user-guide/tasks/use-r-language/)).

- A environment manager.  It can create a virtual environment
  independent from that of the system in order to manage and test
  different version of packages in different virtual environment on
  the same host system.

All the operations are carried out by the command line tool --
`conda`, as well as its GUI tool -- `anaconda-navigator`.

There are over 200 Python packages installed by default during the
installation of Anaconda.  The whole Anaconda directory will occupy
3.4GB.  A much smaller alternative is
[Miniconda](https://conda.io/miniconda.html).  Miniconda only contains
those packages make `conda` work, and is about 300MB after
installation.

`pip install` and `conda install` can be used interchangeably, but use
`conda install` would be better.  Because `conda` package will not
only specify Python depandencies, but also describe system
dependencies to be installed if any.  However, some packages can only
be installed by `pip install`, because no `conda` version is
available.

**Note**: Anaconda would append its own `pip` in the `PATH`, which
will shadow the system's `pip`.  But from Anaconda 2019.03 on, it
won't do that any more so `pip` will be treated like other Python
packages as an independent package in every conda environment.  In
other words, if you are in a conda environment, you can only use `pip
install` when `pip` itself being installed in that very environment.
Thus it won't conflict with the system's `pip` any more.


## Content ##

* [Installation](#installation)
* [Structure of Anaconda installation directory](#structure-of-anaconda-installation-directory)
* [Quick start](#quick-start)
* [Virtual environment](#virtual-environment)
* [Common workflow of developing packages with `conda`](#common-workflow-of-developing-packages-with-conda)
* [Install Anaconda inside Docker](#install-anaconda-inside-docker)


## Installation ##

The Python verison of Anaconda determines the main working environment
when you working with Anaconda, but does not mean you cannot use
another version of Python, because you can use `conda` to create a
virtual environment with a different version of Python from the Python
runing `conda`.  If you are not sure, feel free to choose Anaconda
with Python 3.

See [Installation](https://docs.anaconda.com/anaconda/install/) for
more details on different OS systems.  On Linux, such as Ubuntu, see
[Installing on
Linux](https://docs.anaconda.com/anaconda/install/linux/) for more
details:

1. Download the installer (for example,
   `Anaconda3-2019.03-Linux-x86_64.sh`) from [Anaconda
   Distribution](https://www.anaconda.com/distribution/)
1. Execute the installer.  For example
   ```consolecode
   bash ~/Downloads/Anaconda3-2019.03-Linux-x86_64.sh
   ```

## Structure of Anaconda installation directory ##

It will be installed under `~/anaconda` or `~/miniconda` by default,
and will include these subdirectories:

+ `pkgs`
  - All the packages downloaded and their uncompressed files.  When
    creating a virtual environment, `conda` will make a symbolic link
    to them, so that different version of the same packages can
    co-exist.

+ `envs`
  - This is where all virtual environments are located.  In a `conda`
    virtual environment, `CONDA_PREFIX` denotes the root dir of
    current virtual environment, so if you are in a virtual
    environment called `test`, then `cd $CONDA_PREFIX` will jump to
    `~/anaconda/envs/test`.

+ `bin`, `include`, `lib`, `share`
  - The default environment directories of Anaconda, that is the
    `base` conda environment.  If you create another new virtual
    environment, it will also include these 4 dirs, but may be with
    different versions of the dependencies.


## Quick start ##

```console
$ conda --version     # Check conda version
$ conda update conda  # Update conda

$ # Check available existing virtual environments.  There is a default one called base
$ conda info --envs
$ conda activate <envs-name>      # Switch to the virtual environment <envs-name>
(<envs-name>) $ conda deactivate  # Quit virtual environment <envs-name>

$ # Create a virtual environment as same as conda system environment
$ conda create --name <envs-name>

$ # Create a virtual environment but with Python 3.3
$ conda create --name <envs-name> python=3.3

$ # Create a virtual environment as same as conda system environment,
$ # with pillow and scipy installed.  It is equivalent to:
$ # 
$ #   # Create a virtual environment called <envs-name>
$ #   conda create -n <envs-name>
$ # 
$ #   # Install pillow and scipy inside <envs-name>, make sure scipy version 0.15.0
$ #   conda install -n <envs-name> pillow scipy=0.15.0
$ #
$ conda create --name <envs-name> pillow scipy=0.15.0

$ # Install pillow inside the current virtual environment
$ conda install pillow
$ # Install pillow inside the virtual environment <envs-name>
$ conda install -n <envs-name> pillow
$ # Channels are repos holding packages.  The same package can be found in different 
$ # channels.  The default channel is https://repo.anaconda.com.
$ # If a package is not found in the default channel, you can search the package on 
$ # https://anaconda.org/ and then install it by specifying the channel where it is.
$ conda install -c <channel-name> pillow

$ conda search pillow  # Search for pillow in the conda repo
$ conda list           # List installed packages in current virtual environment
$ conda list -n <envs-name>  # List packages in virtual environment <envs-name>
$ conda list -n <envs-name> pillow  # List the pillow package in <envs-name>
```


## Virtual environment ##

```console
$ # Create a <new-envs> as same as <exist-envs>
$ conda create --name <new-envs> --clone <exist-envs>

$ conda activate <envs-name>  # Switch into virtual environment <envs-name>
$ # Export the config of <envs-name> into environment.yml
$ # You can also run withuot conda activate <envs-name>' by:
$ #     conda env export -n <envs-name> > environment.yml
$ conda env export > environment.yml
$ # Create a new virtual environment as same as described in environment.yml
$ conda env create -f environment.yml

$ conda env remove -n <envs-name>  # Remove virtual environment <envs-name>

$ conda list --revisions  # List the revision history of current virtual environment
2018-11-08 03:08:33  (rev 0)
    +_ipyw_jlab_nb_ext_conf-0.1.0
    +alabaster-0.7.11
    +anaconda-5.3.1

2018-11-26 13:16:44  (rev 1)
     _ipyw_jlab_nb_ext_conf  {0.1.0 -> 0.1.0}
     alabaster  {0.7.11 -> 0.7.12}
     anaconda-client  {1.7.2 -> 1.7.2}
    +_tflow_select-2.3.0
    +absl-py-0.6.1
    +astor-0.7.1
$ # Rollback current environment to version n.  However, a new rollback revison will
$ # be added in the history as well.
$ conda install --revisions n
```

## Common workflow of developing packages with `conda` ##

```console
$ conda create -n <test-envs> <packages-needed>  # Create VE, and install pkgs
$ conda activate <test-envs>  # Swith into the new VE
$ cd /path/to/source/dir  # Go to the dir you will work on
$ python setup.py install  # Install the package you developed into the VE
$ python setup.py develop  # Or install with symbolic link
$ conda deactivate  # Finish testing, quit VE
$ conda remove -n <test-envs> --all  # Remove unused VEs
```


### Reference ###

- [Creating a Conda development environment](https://quantecon.org/wiki-py-conda-dev-env)



## Install Anaconda inside Docker ##

Here is just a try.

```dockerfile
FROM ubuntu:bionic-20181112

# Reference: https://hub.docker.com/r/continuumio/anaconda/~/dockerfile/
RUN apt-get update && \
    apt-get install --no-install-recommends -y wget && \
    wget --quiet https://repo.anaconda.com/archive/Anaconda3-5.3.1-Linux-x86_64.sh -O ~/anaconda.sh && \
    bash ~/anaconda.sh -b -p /opt/conda && \
    rm ~/anaconda.sh && \
    /opt/conda/bin/conda clean -y -a && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate base" >> ~/.bashrc && \
    apt-get remove wget && \
    apt-get autoremove && \
    apt-get clean && \
    rm -rf /var/log/* /var/cache/*

ENV PATH=/opt/conda/bin:${PATH}
```

### Reference ###

- [continuumio/anaconda -- DockerHub](https://hub.docker.com/r/continuumio/anaconda/~/dockerfile/)


## Reference ##

- [Conda Documentation](https://conda.io/docs/)
- [Getting started with conda](https://conda.io/docs/user-guide/getting-started.html)
- [Tasks](https://conda.io/docs/user-guide/tasks/index.html)
- [Tutorials](https://conda.io/docs/user-guide/tutorials/)
- [Command reference](https://conda.io/docs/commands.html)

- [Why you need Python environments and how to manage them with Conda](https://medium.freecodecamp.org/why-you-need-python-environments-and-how-to-manage-them-with-conda-85f155f4353c)
- [Python - Create Your Own Environment using Anaconda](https://wikis.nyu.edu/display/ADRC/Python+-+Create+Your+Own+Environment+using+Anaconda)
- [2-minute recipe: How to rollback your conda environment](https://hackernoon.com/2-minute-recipe-how-to-rollback-your-conda-environment-8521208f9a6c)

