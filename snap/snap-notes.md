# Snap notes #

## Content ##

* [Basic usage](#basic-usage)
* [Advanced Usage](#advanced-usage)
* [Create a Snap package](#create-a-snap-package)
* [Build a MLHUB Snap package](#build-a-mlhub-snap-package)


## Basic usage ##

```console
$ # snapd is a service that deals with snap packages, and by default
$ # installed on Ubuntu 18.04 LTS.
$ sudo apt install snapd

$ # Find a snap package has 'hello' in its name.  Equivalent to:
$ #     $ snap search hello
$ snap find hello

$ # Install a snap package called 'hello'.  Package installation need
$ # root previledge.
$ sudo snap install hello
$ # Install snap package from the 'edge' channel.  Equivalent to:
$ #     $ sudo snap install hello --channel edge
$ # There are 4 channels: stable, candidate, beta and edge
$ sudo snap install hello --edge

$ # List all snap packages installed in the system.
$ snap list

$ # Update all snap packages to their newest version if available.
$ # To update a specific package:
$ #     $ sudo snap refresh hello

$ # Refresh will check if the package available on the tracking
$ # channel, and install the latest version on the tracking channel:
$ #     $ sudo snap refresh hello --edge
$ # and the tracking channel will change upon every refresh with a
$ # specifed channel option.
$ sudo snap refresh

$ # Revert a package to its previous version installed.
$ # Every time you change the intalled version of the package, the
$ # previous installed version of the package is recorded.  However,
$ # the tracking channel remains the same after you revert to a
$ # previous version.
$ sudo snap revert hello

$ # Remove a package.  All package related files are removed.
$ sudo snap remove hello
```

NOTE:
```console
$ sudo snap list
Name                  Version         Rev   Tracking  Publisher   Notes
hello                 2.10            20    stable    canonical✓  -
$ sudo snap install hello --edge
snap "hello" is already installed, see 'snap help refresh'
$ sudo snap refresh hello --edge
hello (edge) 2.10.42 from Canonical✓ refreshed
$ sudo snap list
Name                  Version         Rev   Tracking  Publisher   Notes
hello                 2.10.42         34    edge      canonical✓  -
$ sudo snap refresh hello 
snap "hello" has no updates available
$ sudo snap list
Name                  Version         Rev   Tracking  Publisher   Notes
hello                 2.10.42         34    edge      canonical✓  -
$ sudo snap revert hello 
hello reverted to 2.10
Channel edge for hello is closed; temporarily forwarding to stable.
$ sudo snap list
Name                  Version         Rev   Tracking  Publisher   Notes
hello                 2.10            20    edge      canonical✓  -
$ sudo snap refresh hello 
hello (edge) 2.10.42 from Canonical✓ refreshed
$ sudo snap list
Name                  Version         Rev   Tracking  Publisher   Notes
hello                 2.10.42         34    edge      canonical✓  -
```


### Reference ###

- [Basic Snap Usage](https://tutorials.ubuntu.com/tutorial/basic-snap-usage)


## Advanced usage ##

```console
$ # Every command is a transaction.
$ sudo snap install hello
hello 2.10 from Canonical✓ installed

$ # The history of transactions can be view by snap changes
$ snap changes 
ID   Status  Spawn               Ready               Summary
8    Done    today at 11:26 +08  today at 11:27 +08  Auto-refresh 7 snaps
9    Done    today at 11:38 +08  today at 11:38 +08  Install "hello" snap

$ # Every transaction has a ID, and consists of several steps
$ snap change 9
Status  Spawn               Ready               Summary
Done    today at 11:38 +08  today at 11:38 +08  Ensure prerequisites for "hello" are available
Done    today at 11:38 +08  today at 11:38 +08  Download snap "hello" (20) from channel "stable"
Done    today at 11:38 +08  today at 11:38 +08  Fetch and check assertions for snap "hello" (20)
Done    today at 11:38 +08  today at 11:38 +08  Mount snap "hello" (20)
Done    today at 11:38 +08  today at 11:38 +08  Copy snap "hello" data
Done    today at 11:38 +08  today at 11:38 +08  Setup snap "hello" (20) security profiles
Done    today at 11:38 +08  today at 11:38 +08  Make snap "hello" (20) available to the system
Done    today at 11:38 +08  today at 11:38 +08  Automatically connect eligible plugs and slots of snap "hello"
Done    today at 11:38 +08  today at 11:38 +08  Set automatic aliases for snap "hello"
Done    today at 11:38 +08  today at 11:38 +08  Setup snap "hello" aliases
Done    today at 11:38 +08  today at 11:38 +08  Run install hook of "hello" snap if present
Done    today at 11:38 +08  today at 11:38 +08  Start snap "hello" (20) services
Done    today at 11:38 +08  today at 11:38 +08  Run configure hook of "hello" snap if present

$ # Download the snap files of package instead of installation
$ snap download hello
Fetching snap "hello"
Fetching assertions for "hello"
Install the snap with:
   snap ack hello_20.assert
   snap install hello_20.snap
   
$ # .snap is the snap file of a package, .assert is the assertion or
$ # signature or certificate of a package.
$ ls
hello_20.assert  hello_20.snap

$ # Install from .snap, but be sure you have .assert as well.
$ # Otherwise, you have to use --dangerous if .assert isn't available:
$ #     sudo snap install --dangerous hello_20.snap
$ # Or via --devmode for package development:
$ #     sudo snap install --devmode hello_20.snap
$ # Or provide .assert seperately:
$ #     sudo snap ack hello_20.assert
$ sudo snap install hello_20.snap
```

### Reference ###

- [Advanced Snap Usage](https://tutorials.ubuntu.com/tutorial/advanced-snap-usage)


## Create a Snap package ##

```console
$ # Install snap
$ sudo apt install snapd

$ # Install other necessary dependencies
$ sudo apt install build-essential

$ # Install tools for snap packge creation
$ sudo snap install --classic snapcraft

$ # Make a folder holding all the files for the package
$ mkdir -p /path/to/a/snap/pkg
$ cd path/to/a/snap/pkg

$ # Initialise the package folder.
$ # Afterwards, a folder called snap is created, within which is snapcraft.yaml
$ # snapcraft.yaml defines all meta data about the snap package.
$ # The most important part of snapcraft.yaml:
$ #     name: my-hello
$ #     parts:
$ #       gnu-hello:  # the name is arbitrary and defines the logical parts of a snap package
$ #         source: http://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz  # file source
$ #         plugin: autotools  # snap plugin describes how to build the file source
$ #     apps:  # Expose command
$ #       hello:  # Command name
$ #         command: bin/hello  # Command path relative to snap folder
$ snapcraft init

$ # build snap package after finishing definition of the snap package in snapcraft.yaml
$ snapcraft

$ # Install
$ sudo snap install --devmode my-hello_xx.snap

$ # Invoke the command of the created snap package
$ my-hello.hello
```

### Reference ###

- [Create Your First Snap](https://tutorials.ubuntu.com/tutorial/create-your-first-snap)


## Build a MLHUB Snap package ##

From the reference, it seems there is not a natural way to build a
Snap package for MLHUB, at least some hacks needed.


### Reference ###

+ [Python apps](https://docs.snapcraft.io/python-apps/6741)
  - A pre-defined plugin which can be used for create a python
    package.  Its source code can be found at [snapcraft python plugin -- launchpad](https://git.launchpad.net/snapcraft/tree/snapcraft/plugins/python.py).
    The official Python plugin of snapcraft is a subclass of
    [`BasePlugin`](https://git.launchpad.net/snapcraft/tree/snapcraft/_baseplugin.py)
    and it only supports pip, see
    [snapcraft/plugins/\_python/\_pip.py](https://git.launchpad.net/snapcraft/tree/snapcraft/plugins/_python/_pip.py)

+ [Writing local plugins](https://docs.snapcraft.io/writing-local-plugins/5125)
  - It seems if you want to do some custom operations beyond what the
    pre-defined plugins offer, you have to define your own plugin.

+ [Building your first snap](https://github.com/nextcloud/nextcloud-snap/wiki/Building-your-first-snap)
  - Includes how to add extra files in a snap package.

+ [Snapcraft packaging for Python 3 and 2](https://github.com/jhenstridge/python-snap-pkg)
  - A more sophisticated example of Python snap package.

+ [Bundle RStudio Desktop for Linux as a Flatpak, Snap, or AppImage](https://github.com/rstudio/rstudio/issues/3079)
  - A discussion on putting RStudio into one of the packages.

