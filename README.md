# Creating Debian packages in container

## Overview

Container engines, in particular docker and podman, can be used for Debian
packaging. Building inside a container avoids installing build dependencies on
the host, ensures a clean and reproducible environment, and in case of podman
allows to perform the build as unprivileged user.

Fork of [docker-deb-builder](https://github.com/tsaarni/docker-deb-builder),
created by Tero Saarni.

## Create build environment

The build environment is setup in advance by creating a container image to
speed up builds. It only contains essential build dependencies, like gcc, thus
one can be used to build different Debian packages. For each distribution a
separate build environment needs to be created.

In this example the target is Ubuntu 22.04 and Debian sid, other distributions
can be created by their respective Dockerfile:

    docker build -t container-deb-builder:22.04 -f Dockerfile-ubuntu-22.04 .
    podman build -t container-deb-builder:sid -f Dockerfile-Debian-sid-unstable .

The image name (`container-deb-builder:22.04`) is later used while building a
Debian package.

## Building packages

First download or git clone the source code of the package to build:

    git clone ... ~/my-package-source

The source code should contain subdirectory called `debian` with at
least a minimum set of packaging files: `control`, `copyright`,
`changelog` and `rules`.

Run the build script to see its usage:

    $ ./build -h
    usage: build [options...] SOURCEDIR
    Options:
      -i IMAGE     Name of the docker image (including tag) to use as package build environment.
      -o DIR       Destination directory to store packages to.
      -d DIR       Directory that contains other deb packages that need to be installed before build.
      -p profiles  Specify the profiles to build (e.g. nocheck). Takes a comma separated list.
      -C           Use ccache to cache compiled objects.
      -L           Run Lintian after a successful build.

To build Debian packages run following commands:

    # create destination directory to store the build results
    mkdir output

    # build package from source directory
    ./build -i container-deb-builder:22.04 -o output ~/my-package-source

After a successful build the build results will be copied from the container
into the `output` directory. The container itself is discarded.

Sometimes builds might require dependencies that cannot be installed with
`apt-get build-dep`, e.g. when the required version of the dependency is not
yet available.  Those can be installed into the build environment by passing
the option `-d DIR`, where *DIR* is a directory with `*.deb` files in it.

    ./build -i container-deb-builder:22.04 -o output -d dependencies ~/my-package-source

## Limitations

* Since the package specific build dependencies are installed into the
  container at runtime, the container, and therefore the build process, has
  network access.
