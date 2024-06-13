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

.. code-block:: console

  source spack/share/spack/setup-env.sh
  apptainer pull docker://rockylinux/rockylinux:9

This will download a basic container that runs on Rocky Linux 9 rather than Ubuntu that your VM is running.

Once the container is finished downloading we will look at the differences between the two containers. Before starting the container run the command

.. code-block:: console

  tar --version

Now lets start a session within the container and run the command again:

.. code-block:: console

  apptainer shell rockylinux_9.sif
  tar --version

Note that the container has a different version of tar than the main operating system has. This can be used to build an entire environment with the exact versions of software and libraries needed to execute your research software.

Additionally commands to containers can be passed non-interactively. For HPC systems, when submitting jobs this will be the main method of calling containers within job scripts:

.. code-block:: console

  apptainer exec rockylinux_9.sif tar --version


Leveraging Spack 
------------------ 

Using Spack we can simplify the build process of environments for containers substantially. Spack has the ability to write an entire build file for a new container from a simple YAML list of packages that Spack can provide. Here we will set up a build for a simple container with a single package using Spack's containerize function.

First we set up the environment for spack and create a new spack.yaml file to read from

.. code-block:: console

  $ mkdir apptainer
  $ cd apptainer
  $ . spack/share/spack/setup-env.sh 
  $ nano spack.yaml

Inserting this code into the spack.yaml file will tell Spack we want 

.. code-block:: console
  
  spack:
   specs:
    - ffmpeg
   container:
    format: singularity

Now that we have the packages all loaded we start up apptainer and run the containerize function to make a build definitions file

.. code-block:: console

  $ spack load apptainer
  $ spack containerize > spack-user-ffmpeg.def
  $ apptainer build spack-user-ffmpeg.sif spack-user-ffmpeg.def

Spack will then build from source everything needed for the container and package it within the output .sif file.

Using Apptainer Containers
-----------------------------

.. code-block:: console

  $ apptainer exec --fakeroot spack-user-ffmpeg.sif ffmpeg -h

Here we see that the ffmpeg package is installed and ready for use within the container we built.


Building Apptainer containers from scratch
=============================================

In some cases the entire set of software you need to build a container is not available in Spack. This can be particularly true if you have self compiled code that needs to be pre-built for your jobs to execute functions from. In that case we can build a Apptainer build file and use that to construct our environment. Lets break down the key components of a build file and then put them together to build an image.

Apptainer Image Header
^^^^^^^^^^^^^^^^^^^^^^^

Every build file starts with a base image and a location to pull the image from. In our case lets look at a basic Ubuntu image as the starting point

.. code-block:: console

  Bootstrap: docker
  From: ubuntu:22.04

This tells us we want a container from DockerHub from Ubuntu with the release 22.04. More complex build files such as the ones generated by Spack will also include a 'Stage' command to allow you to break up compiling and building the container into multiple stages to reduce container size. For this demo we will be working just with a single stage container.

Next we will define our environment variables that will be set up each time the container launches. This is very useful if you have a complex install path and would like it to be set up for easy execution from the command line.

.. code-block:: console

  %environment
  export PATH=/opt/new_software/bin:${PATH}
  export EXAMPLE_VAR=23

Finally we have the main block for the build file: 'post'. This block defines all of the commands we want to run to build up the environment and install software. Here we can place commands to set up our software in `/opt/new_software/bin` and ensure it is ready to go when the container finishes building.

.. code-block:: console

  %post
  apt-get update && apt-get install -y --no-install-recommends  wget tar zip man git gcc
  mkdir -p /opt/new_software/bin
  cd /opt/new_software/bin
  wget --no-check-certificate https://github.com/ruanyf/simple-bash-scripts/raw/master/scripts/color.sh
  chmod +x color.sh

This puts a simple bash script into our path. Now lets finish off and build the container to see how it executes. Please use whatever you named the build file in place of 'my_container.def'

.. code-block:: console

  apptainer build my_container.sif  my_container.def

Now finally we can execute the container built and see the colored output from the script we added.

.. code-block:: console

  apptainer exec my_container.sif color.sh
  
Sandboxing and 'Editing' Containers
==========================================

In some cases you may want to test changes to a container or work on developing a container on the fly without having to rebuild the container each time you make changes. An option is available in Apptainer to create a sandbox of an existing containter. Rather than a compact .sif file sandboxing creates a set of directories that work as a container environment. This allows writing and making changes to the environment for testing.

.. code-block:: console

  apptainer build --sandbox my_container/ my_container.sif

This will build our previous container into a sandbox directory called 'my_container'. We can connect to the sandbox directory and run commands to edit the environment with:

.. code-block:: console

  apptainer shell --writable my_container/
  
Any changes made in this shell to the environment, such as installing packages, will persist between sessions and be written into the directories within the sandbox. This can be very useful for testing what you would like to write into your .def file for future builds or changes to the container. For production work it is generally much better to use a .sif style container for execution as it is both compresses and written as a single file which makes it much more portable and efficent to use on most filesystems. 

It is possible to convert a sandbox back into a .sif file, but do note that in doing so there is no record of what changes were made to the .sif. For reproducibility it its far better to add the changes made to a .def file as a record. To convert back from a writable container into a .sif you may use:

.. code-block:: console

  apptainer build my_edited_container.sif my_container/



==========================================
Advanced Topics with Containers
==========================================

Beyond the basics of software building here there are several other more complicated uses of containers that are useful to discuss for HPC usage. These topics do require a basic understanding of containers and their build process that we covered in the morning session. 


Python Environments in Containers
===================================

On HPC systems it is common to build virtual environments for python workflows that include several packages. Rebuilding these environments to be the same on multiple systems can be challenging as well as time consuming. Containers can help alleviate this work by building the environment once and making it portable within a single container file.

For our example we will build a container using the 'pandas_environment.txt' file that contains a list of all of the python packages for a conda virtual environment to use the Pandas data analysis library. This example can be extended to any other conda environment as well by exporting or building a requirements file and performing a similar operation on building the container.

To start off we want to work with a container that has the conda software already installed. To do this, rather than starting from a blank Ubuntu image we can actually use a prebuilt image that has miniconda3 already set up.

.. code-block:: console

  Bootstrap: docker
  From: continuumio/miniconda3

We next want to bring in our environment file so it is avaiable during the build process so it can be used as a lookup for what packages conda will look to install. The %files tag will copy in any file specified from the local system into the build process so that it can be added to a final image. This is a good way to import source code for self-compiled research work as well.

.. code-block:: console

  %files
      pandas_environment.txt

Finally we will want to set up the environment and call the actual build process for the conda installer. Our %environment information is used to ensure that when we launch the container after it is built we have the virtual environement inside already loaded and ready to make python calls against. This requires a bit of extra setup in our %post section to ensure that it is easy for conda to activate the new environment we created.

.. code-block:: console

  %environment
      source /opt/etc/bashrc
      conda activate singularityenv

  %post
      /opt/conda/bin/conda config --env --add channels conda-forge
      /opt/conda/bin/conda env create -n singularityenv --file pandas_environment.txt
      conda init bash
      mkdir -p /opt/etc
      cp ~/.bashrc /opt/etc/bashrc

Putting this all together into a def file we can once again call `apptainer build` to construct a new .sif file. Once it is finished we can use `apptainer shell` or `apptainer exec` to make python calls using the containers installation of python and Pandas




MPI Workloads and Containers
=============================

MPI is a common interface for high performance computing allowing software to make use of multiple nodes for single problems by spreading the memory and computing workload over large numbers of CPUs and sets of system memory. Containers can also be used in these instances but it is important to understand the style and version of MPI interfaces used by the HPC system you will be operating on. 

Ideally when constructing your container the type of MPI software in the container should be similar or identical to the one used on the HPC system for best performance. In many cases it is worthwhile to reach out to the system administration team of the HPC system or review their documentation on how best to use containers with MPI on their system.

Our demonstration cluster has a basic MPI framework installed so we will not be able to test a container with MPI capabilities but we will cover some of the usage basics.

.. code-block:: console

  mpiexec apptainer exec my_container.sif mpi-software --option 1 --setting 2 --input datafile.in


GPU usage with Containers
===========================

GPUs have become an increasingly powerful and common tool to use with research computing. AI and machine learning software are extremely common users of GPUs but other software is beginning to make use of the accelerated capabilities of GPU processing power as well. Containers can also interface with GPUs for their software as well.

Although we did not have an example to show building a GPU container they can be built much the same as above. Depending on the type of GPU you are utilizing you will need to include the CUDA or ROCm libraries in the container for your software to function as well as make an additional flag during the `apptainer exec` or `apptainer shell` commands to import the GPU devices into the container. These can be activated by using the `--nv` or `--rocm` flags respectively depending on the GPU hardware type.

.. code-block:: console

  apptainer exec --nv my_container.sif python3 my_pytorch.py
  
  
Multi-stage Builds
===================

To reduce sizes of the final containers and break builds up into multiple layers the 'Stage' tag can be used in container build files. Spack uses this by default with one stage being the build process where sources are installed and built and the second stage moves all of the binaries and required libraries to a new clean container and sets up the environment there.

In the header for a container there is the option to provide the `Stage:` tag to define and break up a single container build into multiple sections of building. This creates a layered approach where early stages can be used to build software dependencies that need substantial additional packages that are not needed in the final output image.

.. code-block:: console

  Bootstrap: docker
  From: continuumio/miniconda3
  Stage: 
  
  
Hardware Architecture Caveats
===============================

Although containers can create portable software environments, when making your software portable via containers it is important to know the limitations of the software built within the container as well. Many times when software is compiled from source the software will look to optimize for the CPU architecture that is available on the current system. When copying the container to another system it may be that the hardware instructions in the compiled code are not supported on the CPU itself. This will often lead to an 'Instruction Error' being reported and the code failing to start.

Depending on how your software is built it may be possible to over-ride the default of build arctitecture to target a more limited processer instruction set to make your compiled code more portable across multiple arcitectures. Review your software build instructions or compiler flags with 'gcc' or other compilers for how to accomplish this.

