
======================================================================
Welcome to the UBC ARC 2024 Summer School Container Workshop: Part 1
======================================================================

In this morning's workshop we will explore the basics of containers for research computing and how we can create a portable software environment with them.



About Containers
-----------------

Containerized software is becoming more prevelant throughout the computing landscape and that includes research computing. Have you ever had an environment that you have spent hours installing and preparing and then needed to turn around and have a colleague need to replicate it, or worse, you need to migrate to an entirely new system? Containers are prefect for this sort of scenario. If you build it once in a container, the file can be brought and shared to any system that runs a container framework and launch it to run software without worrying about the environment on the local machine.

A container functions as effectively an isolated operating system on a node while it is running. Commands and software executed within the container will therefore run using this isolated system. This has many, many applications but for today we will explore how this can be applied to research workloads.

Two common frameworks for containers in research computing are:

* Docker
* Apptainer/Singularity

We will focus on using Apptainer but note that Docker containers are also supported by Apptainer and infact will be the basis of several containers we will be building. Containers generally follow an open standard maintained by the OCI or Open Container Initative. (https://opencontainers.org/) This allows for containers to be shared across platforms and maintian most of their critical features. 

A general rule with containers is they are built in layers. Each stage of changes adds another layer ontop of an already existing container. This is both useful for simplifying complex environments as well as documenting the changes made to a container to allow them to be more easily reproduced.


General Rules on Building and Using Containers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

With Apptainer, once built container files are immutable. This ensures that each time you run the container nothing will have changed between uses with the environment and even if it is copied to other systems. This does mean however that if a container file is built and needs to be modified, the container will need to be rebuilt. Apptainer does provide a 'sandbox' style environment for building and modifying containers on the fly that we will cover later during this session.

In the case of both Apptainer and Docker a build file can be used to construct and script the installation of software within the container while building. These files also can serve as a record of the steps taken during building for auditing the software environment or to allow other colleagues to rebuild and modify the container without adding another layer ontop of the existing container.


Building Apptainer Containers
==============================

On our workshop cluster we have pre-installed a number of packages for utilizing and creating containers. The primary software is Apptainer which is the main framework for the build and execution process. We have also installed the Spack environment that contains a tool to easily build container definition files for an environment of software built from source. For building on a local machine you will need at least the Apptainer envrionment. Installation instructions can be found here: https://apptainer.org/docs/admin/main/installation.html#install-from-pre-built-packages 


Downloading Pre-built Containers
---------------------------------

Sometimes everything you already need is available in a container online. This can save time on building an environment by simply pulling a container that is ready for your use. The most common repository for containers is Docker Hub : <https://hub.docker.com>. This website hosts a variety of Docker containers that are both uploaded by users and organizations and are freely able to be pulled and run on local machines with Apptainer.

To start off we will run the following command:

.. code-block:: bash

  module load gcc apptainer
  apptainer pull docker://rockylinux/rockylinux:9

This will download a basic container that runs on Rocky Linux 9 rather than Ubuntu that your VM is running.

Once the container is finished downloading we will look at the differences between the two containers. Before starting the container run the command

.. code-block:: bash

  tar --version

Now lets start a session within the container and run the command again:

.. code-block:: bash

  apptainer shell rockylinux_9.sif
  tar --version

Note that the container has a different version of tar than the main operating system has. This can be used to build an entire environment with the exact versions of software and libraries needed to execute your research software.

Additionally commands to containers can be passed non-interactively. For HPC systems, when submitting jobs this will be the main method of calling containers within job scripts:

.. code-block:: bash

  apptainer exec rockylinux_9.sif tar --version


Leveraging Spack 
------------------ 

Using Spack we can simplify the build process of environments for containers substantially. Spack has the ability to write an entire build file for a new container from a simple YAML list of packages that Spack can provide. Here we will set up a build for a simple container with a single package using Spack's containerize function.

First we set up the environment for spack and create a new spack.yaml file to read from

.. code-block:: bash

  mkdir apptainer-ffmpeg
  cd apptainer-ffmpeg
  . /project/spack/share/spack/setup-env.sh 
  nano spack.yaml

Inserting this code into the spack.yaml file will tell Spack we want 

.. code-block:: yaml
  
  spack:
   specs:
    - ffmpeg
   container:
    format: singularity

Now that we have the packages all loaded we start up apptainer and run the containerize function to make a build definitions file

.. code-block:: bash

  module load gcc apptainer
  spack containerize > spack-user-ffmpeg.def
  apptainer build spack-user-ffmpeg.sif spack-user-ffmpeg.def

Spack will then build from source everything needed for the container and package it within the output .sif file.

Using Apptainer Containers
-----------------------------

.. code-block:: bash

  apptainer exec --fakeroot spack-user-ffmpeg.sif ffmpeg -h

Here we see that the ffmpeg package is installed and ready for use within the container we built.


Building Apptainer containers from scratch
=============================================

In some cases the entire set of software you need to build a container is not available in Spack. This can be particularly true if you have self compiled code that needs to be pre-built for your jobs to execute functions from. In that case we can build a Apptainer build file and use that to construct our environment. Lets break down the key components of a build file and then put them together to build an image.

Apptainer Image Header
^^^^^^^^^^^^^^^^^^^^^^^

Every build file starts with a base image and a location to pull the image from. In our case lets look at a basic Ubuntu image as the starting point

.. code-block:: singularity

  Bootstrap: docker
  From: ubuntu:22.04

This tells us we want a container from DockerHub from Ubuntu with the release 22.04. More complex build files such as the ones generated by Spack will also include a 'Stage' command to allow you to break up compiling and building the container into multiple stages to reduce container size. For this demo we will be working just with a single stage container.

Next we will define our environment variables that will be set up each time the container launches. This is very useful if you have a complex install path and would like it to be set up for easy execution from the command line.

.. code-block:: singularity

  %environment
  export PATH=/opt/new_software/bin:${PATH}
  export EXAMPLE_VAR=23

Finally we have the main block for the build file: 'post'. This block defines all of the commands we want to run to build up the environment and install software. Here we can place commands to set up our software in `/opt/new_software/bin` and ensure it is ready to go when the container finishes building.

.. code-block:: singularity

  %post
  apt-get update && apt-get install -y --no-install-recommends  wget tar zip man git gcc
  mkdir -p /opt/new_software/bin
  cd /opt/new_software/bin
  wget --no-check-certificate https://github.com/ruanyf/simple-bash-scripts/raw/master/scripts/color.sh
  chmod +x color.sh

This puts a simple bash script into our path. Now lets finish off and build the container to see how it executes. Please use whatever you named the build file in place of 'my_container.def'

.. code-block:: bash

  apptainer build my_container.sif  my_container.def

Now finally we can execute the container built and see the colored output from the script we added.

.. code-block:: bash

  apptainer exec my_container.sif color.sh
  
Sandboxing and 'Editing' Containers
==========================================

In some cases you may want to test changes to a container or work on developing a container on the fly without having to rebuild the container each time you make changes. An option is available in Apptainer to create a sandbox of an existing containter. Rather than a compact .sif file sandboxing creates a set of directories that work as a container environment. This allows writing and making changes to the environment for testing.

.. code-block:: bash

  apptainer build --sandbox my_container/ my_container.sif

This will build our previous container into a sandbox directory called 'my_container'. We can connect to the sandbox directory and run commands to edit the environment with:

.. code-block:: bash

  apptainer shell --writable my_container/
  
Any changes made in this shell to the environment, such as installing packages, will persist between sessions and be written into the directories within the sandbox. This can be very useful for testing what you would like to write into your .def file for future builds or changes to the container. For production work it is generally much better to use a .sif style container for execution as it is both compresses and written as a single file which makes it much more portable and efficent to use on most filesystems. 

It is possible to convert a sandbox back into a .sif file, but do note that in doing so there is no record of what changes were made to the .sif. For reproducibility it its far better to add the changes made to a .def file as a record. To convert back from a writable container into a .sif you may use:

.. code-block:: bash

  apptainer build my_edited_container.sif my_container/


