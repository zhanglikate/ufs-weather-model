# Containerization of the ufs-weather-model

This directory contains the information to containerize the ufs weather model code.

## Guide
1. [Background](#background)
2. [Running the UFS](#running)
3. [GNU Compilers on Ubuntu 20.04](#ubuntu)
4. [GNU Compilers on Centos 8](#centos)
5. [Intel Compilers on Centos 7](#intel-centos)
6. [Using Singularity](#singularity)
7. [On the Cloud](#cloud)

## Background {#background}

In general there are 4 stages to build the complete UFS Weather model in a container.  The Intel version has an extra step, making 5.  The stages are as follows

1. Build NCEPLIBS-external
2. Build NCEPLIBS-internal
3. Build UFS Weather Model
4. Build 'Lite' container
   > This stage builds a clean container without all the build tools. During the build process, it will copy only the necessary libraries needed to run the UFS.  The resulting container is significantly smaller in size and easier to copy and move around.

These stages could be combined into a single Dockerfile, but by keeping them separate, it allows a developer to more easily modify various components of the model and test the results.

## Running UFS in a Container {#running}

Once you have completed the build process you are able to run the UFS Weather Model within a container.  You can use either the full container with all the build tools, or the 'lite' container with only the necessary runtime components.  This example uses the 'lite' container.

1. Download and unpack the simple test case following step 3 [here](https://ftp.emc.ncep.noaa.gov/EIB/UFS/simple-test-case.tar.gz)

2. Run the UFS container mounting the simple test case as a volume

`
docker run --rm -it -v /data:/data org/ufs-ubuntu20-develop
`
> Important
> 
#### Description of Arguments

*--rm*
: Removes this image after the container exits

*-it*
: Runs the container in an interactive mode with terminal access

*-v /data:/data*
: Mounts your local directory /data as /data within the running image.  This is your <working_directory> where you downloaded the simple test case data

*org/ufs-ubuntu20-develop*
: The name of the container you wish to run

## GNU Compilers on Ubuntu 20.04 {#ubuntu}

To build the UFS Weather Model using GNU Compilers based on Ubuntu 20.04 image

### Building NCEPLIBS-External {#u20-nceplibs-internal}

within the docker directory run the following command:

`
docker build --pull --rm --build-arg NCEPLIBS_VERSION=develop -t org/ufs-ubuntu20-ncepext -f Dockerfiles/gnu.ubuntu20/Dockerfile-ncepext .
` 

Upon success, this will generate a Docker image with the tag _org/ufs-ubuntu20-ncepext_

#### Description of arguments 
*--pull*
: Instructs docker to pull the latest version of the base OS. In this case ubuntu:20.04

*--rm*
: Removes intermediate containers after a successful build. It helps keep the docker installation clean

*--build-arg NCEPLIBS_VERSION=develop*
: Specifies to the docker build script to use the 'develop' tag/branch of the NCEPLIBS-external repository

*-t org/ufs-ubuntu20-ncepext* 
: Specifies our local tag for this container after a successful build.  This is tag is re-used by the NCEPLIBS-internal build process. You may change to meet your needs, remember this tag for the [next step](#u20-nceplibs-internal)

*-f Dockerfiles/gnu.ubuntu20/Dockerfile-ncepext*
: Specifies the docker file used to build this image

*.*
: Build context to use, a single period '.' means use the current directory

### Building NCEPLIBS-Internal {#u20-nceplibs-internal}

within the docker directory run the following command:

`
docker build --rm --build-arg NCEPLIBS_VERSION=develop --build-arg NCEP_EXTERNAL="org/ufs-ubuntu20-ncepext:latest" -t org/ufs-ubuntu20-nceplibs -f Dockerfiles/gnu.ubuntu20/Dockerfile-nceplibs .
` 

Upon success, this will generate a Docker image with the tag _org/ufs-ubuntu20-nceplibs_

#### Description of arguments 
*--rm*
: Removes intermediate containers after a successful build. It helps keep the docker installation clean

*--build-arg NCEPLIBS_VERSION=develop*
: Specifies to the docker build script to use the 'develop' tag/branch of the NCEPLIBS-external repository

*--build-arg NCEP_EXTERNAL=org/ufs-ubuntu20-ncepext:latest*
: Specifies the NCEPLIBS_external image to base this build on, use the tag you specified in the [previous step](#u20-nceplibs-external) 

*-t org/ufs-ubuntu20-nceplibs* 
: Specifies our local tag for this container after a successful build.  This is tag is re-used by the UFS build process. You may change to meet your needs, remember this tag for the [next step](#u20-ufs)

*-f Dockerfiles/gnu.ubuntu20/Dockerfile-nceplibs*
: Specifies the docker file used to build this image

*.*
: Build context to use, a single period '.' means use the current directory

### Building UFS Weather Model {#u20-ufs}

within the docker directory run the following command:

`
docker build --rm --build-arg --build-arg NCEP_INTERNAL="org/ufs-ubuntu20-nceplibs:latest" --build-arg UFS_VERSION=develop -t org/ufs-ubuntu20-develop-full -f Dockerfiles/gnu.ubuntu20/Dockerfile-ufs .
` 

Upon success, this will generate a Docker image with the tag _org/ufs-ubuntu20-develop-full_

#### Description of arguments 
*--rm*
: Removes intermediate containers after a successful build. It helps keep the docker installation clean

*--build-arg UFS_VERSION=develop*
: Specifies to the docker build script to use the 'develop' tag/branch of the NCEPLIBS-external repository

*--build-arg NCEP_INTERNAL=org/ufs-ubuntu20-nceplibs:latest*
: Specifies the NCEPLIBS_internal image to base this build on, use the tag you specified in the [previous step](#u20-nceplibs-internal) 

*-t org/ufs-ubuntu20-develop-full* 
: Specifies our local tag for this container after a successful build.  This is tag is re-used by the UFS Lite build process. You may change to meet your needs, remember this tag for the [next step](#u20-ufs-lite)

*-f Dockerfiles/gnu.ubuntu20/Dockerfile-ufs*
: Specifies the docker file used to build this image

*.*
: Build context to use, a single period '.' means use the current directory

### Building UFS Weather Model Lite Container {#u20-ufs-lite}

within the docker directory run the following command:

`
docker build --build-arg --build-arg UFS_CONTAINER="org/ufs-ubuntu20-develop-full:latest" --rm -t org/ufs-ubuntu20-develop -f Dockerfiles/gnu.ubuntu20/Dockerfile-ufs-lite .
` 

Upon success, this will generate a Docker image with the tag _org/ufs-ubuntu20-develop_

#### Description of arguments 
*--rm*
: Removes intermediate containers after a successful build. It helps keep the docker installation clean

*--build-arg UFS_CONTAINER=org/ufs-ubuntu20-develop-full*
: Specifies the UFS_CONTAINER image to base this build on, use the tag you specified in the [previous step](#u20-ufs) 

*-t org/ufs-ubuntu20-develop* 
: Specifies our local tag for this container after a successful build.  This is tag is re-used by the UFS Lite build process. You may change to meet your needs, remember this tag for the [next step](#u20-ufs-lite)

*-f Dockerfiles/gnu.ubuntu20/Dockerfile-ufs-lite*
: Specifies the docker file used to build this image

*.*
: Build context to use, a single period '.' means use the current directory
## GNU Compilers on Centos 8 {#centos}
> TODO
## Intel Compilers on Centos 7 {#intel-centos}
> TODO
## Using Singularity {#singularity}
> TODO
## On the cloud {#cloud}

> TODO