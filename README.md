# Beremiz public distribution #

This repository holds recipes and patches to build the Beremiz related 
bundles, packages, installers and images available on 
[Beremiz website](http://www.beremiz.org/).

Patches have same license as projects beeing patched. For other files,
unless made explicit in file header, GPLv3 applies.


## Prerequisites ##

A usable Docker is required. Docker is used to ensure a reproducible build
environment. 

Other containerization/virtualization could be used to achieve the same effect. 

Reference build environment is obtained by applying [provision_focal64.sh](provision_focal64.sh) on Ubuntu 20.04 amd64.

## Preparing source ##

The top Makefile expects source code of Beremiz, Matiec and CanFestival to be 
sibling to beremiz_public_dist directory, in the 'source directory'.
For example :

```
~/src
     /beremiz
     /beremiz_public_dist
     /canfestival
     /matiec
```

If any of Beremiz, Matiec or CanFestival repositories are missing, they will be
cloned aside beremiz_public_dist from https://hg.beremiz.org .

Intermediate and final results are all placed in a single 'build directory'
during build process.

## Windows installer Build ##

### Prepare Docker image ###

The script [rebuild_docker.sh](rebuild_docker.sh) is used for that purpose.

Synopsis:

    ./rebuild_docker.sh [build directory]

Example:

```
#!sh
mkdir ~/src ~/build
cd ~/src
hg clone https://hg.beremiz.org/beremiz_public_dist
cd beremiz_public_dist
./rebuild_docker.sh
```

Source and Build Volumes :
 
 Role             | Docker Volume     | Host path in example | Rationale
------------------|-------------------|----------------------|------------------------------------------------
 source directory | /home/devel/src   | ~/src                | Always relative to CWD : "../"
 build directory  | /home/devel/build | ~/build              | First argument to rebuild_docker.sh or ~/build

'build directory' can be specified as absolute path argument of rebuild_docker.sh.
If not specified it defaults to ~/build

```
#!sh
./rebuild_docker.sh ~/build_next
```

Note: 'build directory' is created if not already existing.

### Build Windows Installer ###

The script [build_in_docker.sh](build_in_docker.sh) is used for that purpose.

```
#!sh
./build_in_docker.sh 
```

Note: 'build directory' must exist before calling build_in_docker.sh, otherwise
Docker will create it as root user. Be sure to create it again if you delete it.

Resulting installer is Beremiz-1.2.exe in 'build directory'. 

### Manhole ###

Once Docker image ready, the scripts [enter_docker.sh](enter_docker.sh) and
[enter_docker_as_root.sh](enter_docker_as_root.sh) let you issue build 
commands, or install additional packages.

[clean_docker_container.sh](clean_docker_container.sh) and [create_docker_container.sh](create_docker_container.sh) and 
[clean_docker_image.sh](clean_docker_image.sh) and [build_docker_image.sh](build_docker_image.sh) are invoked by 
[rebuild_docker.sh](rebuild_docker.sh). 

## Snap Package Build ##

snapcraft.yaml does not rely on HG or GIT repos, but refers to local sources, this is responsibility of caller to prepare sources for snapcraft.

Reasons for this are :
 - no parameters or conditional variables exist to tell snapcraft.yaml where to get the source from
 - building from local source should be always possible
 - revisions.txt must be updated in any case so that it is always possible to know what version wqas used to build resulting package
 - when Snap is built from GitHub actions, Makefile isn't used at all, and GitHub action workflow organize sources and revisions.txt on its own.

In the end, instead of just calling `snapcraft` to get a snap package, it is more complicated:

```
#!sh
mkdir ~/src ~/build
cd ~/src
hg clone https://hg.beremiz.org/beremiz_public_dist
cd ~/build
make -f ~/src/beremiz_public_dist/Makefile DIST=snap

```

