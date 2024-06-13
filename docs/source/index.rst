Welcome to the BCNET 2024 Spack Container Workshop
=======================================================

In this workshop we will guide you through the process of using spack
to build software stacks. First log into our virtual machine
and install spack. Then a deep dive into spack focusing on the 
power of various specs syntax and the flexibility it gives
to users. We will cover the ``spack spec``/``spack install`` for 
installing, the ``spack find``/``spack list`` command for viewing 
installed packages and the ``spack uninstall`` for uninstalling packages. 
Next a section on how to manage compilers with Spack paying close attention 
while using Spack-built compilers within Spack. Then we will cover 
custom build scripts for managing complex software stacks as well lessons
learned using spack. Finally we will take everything we learned using spack
and the package singularity / apptainer to create a singularity container
and run this container.

We will include a few output from the commands demonstrated, to save time
we will frequently call attention to only small portions of
that output.

----------------------
About Spack & Credits 
----------------------

Spack is a package management tool designed to support multiple versions and configurations 
of software on a wide variety of platforms and environments. It was designed for large 
supercomputing centers, where many users and application teams share common installations 
of software on clusters with exotic architectures, using libraries that do not have a 
standard ABI. Spack is non-destructive: installing a new version does not break existing 
installations, so many configurations can coexist on the same system.

Most importantly, Spack is simple. It offers a simple spec syntax so that users can specify 
versions and configuration options concisely. Spack is also simple for package 
authors: package files are written in pure Python, and specs allow package authors to maintain 
a single file for many different builds of the same package.

A big thanks to the Spack team for a great Spack tutorial for references. 

Full citation: Todd Gamblin, Gregory Becker, Massimiliano Culpo, Tamara Dahlgren, Adam J. 
Stewart, and Harmen Stoppels. Managing HPC Software Complexity with Spack. 
Supercomputing 2022 (SC’22). Dallas, TX, November 13, 2022.

https://spack-tutorial.readthedocs.io/en/latest/

----------------
Installing Spack
----------------

Spack works out of the box. Simply clone Spack to get going. We will
clone Spack and immediately check out the most recent release, v0.21.

Now let's install spack inside of our training docker environment
  
.. code-block:: console

  $ docker run -it ghcr.io/spack/tutorial:sc23
  $ git clone -c feature.manyFiles=true https://github.com/spack/spack.git

.. code-block:: console

  Cloning into '/home/spack1/spack'...
  remote: Enumerating objects: 403295, done.K
  remote: Counting objects: 100% (235/235), done.K
  remote: Compressing objects: 100% (147/147), done.K
  remote:nTotale4032959(delta993),4reused,1817(deltaB60),0pack-reused 403060K
  Receiving objects: 100% (403295/403295), 203.42 MiB | 39.28 MiB/s, done.
  Resolving deltas: 100% (162372/162372), done.

Next, add Spack to your path. Spack has some nice command-line
integration tools, so instead of simply appending to your ``PATH``
variable, source the Spack setup script.

.. code-block:: console

  $ . spack/share/spack/setup-env.sh

Ready to go!

-----------------
Inside Spack
-----------------

The ``spack`` command will prompt a feature rich list of common spack commands. 

.. code-block:: console

  $ spack

.. code-block:: console

  A flexible package manager that supports multiple versions,
  configurations, platforms, and compilers.
  
  These are common spack commands:
  
  query packages:
  list                  list and search available packages
  info                  get detailed information on a particular package
  find                  list and search installed packages
  
  build packages:
  install               build and install packages
  uninstall             remove installed packages
  gc                    remove specs that are now no longer needed
  spec                  show what would be installed, given a spec
  
  configuration:
  external              manage external packages in Spack configuration
  
  environments:
  env                   manage virtual environments
  view                  project packages to a compact naming scheme on the filesystem.
  
  create packages:
  create                create a new package file
  edit                  open package files in $EDITOR
  
  system:
  arch                  print architecture information about this machine
  audit                 audit configuration files, packages, etc.
  compilers             list available compilers
  
  user environment:
  load                  add package to the user environment
  module                generate/manage module files
  unload                remove package from the user environment
  
  optional arguments:
  --color {always,never,auto}
                        when to colorize output (default: auto)
  -V, --version         show version number and exit
  -h, --help            show this help message and exit
  -k, --insecure        do not check ssl certificates when downloading
  
  more help:
  spack help --all       list all commands and options
  spack help <command>   help on a specific command
  spack help --spec      help on the package specification syntax
  spack docs             open https://spack.rtfd.io/ in a browser

----------------------
Spack Common Commands
----------------------

The ``spack list`` command shows available packages to install.

.. code-block:: console

  $ spack list --help

Some example query strings for fun.

.. code-block:: console

  $ spack list
  $ spack list 'py-*'
  $ spack list 'py-python*'
  $ spack list '*lib'
  $ spack list 'mpi'
  
The ``spack versions`` command list available versions of a package.

.. code-block:: console

  $ spack versions --help
  $ spack versions tcl
  
The ``spack find`` command shows installed packages / version / compiler used.

.. code-block:: console

  $ spack find --help
  $ spack find 
  
The ``spack spec`` command shows what would be installed, given a spec.

.. code-block:: console

  $ spack spec --help
  $ spack spec -I tcl

The ``spack install`` command will build and install packages.

.. code-block:: console

  $ spack install --help
  $ spack install tcl
  
The ``spack uninstall`` command will remove installed packages.

.. code-block:: console

  $ spack uninstall --help
  $ spack uninstall tcl
  
-----------------------------------------
Spack Install / Uninstall / Build Caches
-----------------------------------------

Lets start with a simple package install of tcl ``spack install``.

.. code-block:: console

  $ spack spec -I  tcl
  
.. code-block:: console

  $ spack spec -I  tcl
  Input spec
  --------------------------------
  -   tcl
  
  Concretized
  --------------------------------
  -   tcl@8.6.12%gcc@7.5.0 build_system=autotools arch=linux-ubuntu18.04-skylake_avx512
  [+]      ^zlib@1.2.13%gcc@7.5.0+optimize+pic+shared build_system=makefile arch=linux-ubuntu18.04-skylake_avx512

You will see the packages needed as well the package requested / version / compiler version. 

lets go ahead and install tcl.

.. code-block:: console

  $ spack install tcl

Now lets start to add custom search strings and flags to our install specifications ``spec``. 
Always use the ``spack spec -I`` command to spec out the install before you do the final install.

first lets get some info the htop package.

.. code-block:: console

  $ spack info htop
 
In one command you get the description,homepage,versions,variant flags, dependencies and more.

Lets spec out version 3.2.0, disable hwloc and enable debug

.. code-block:: console

  $ spack spec -I htop@3.2.0
  $ spack spec -I htop@3.2.0 ~hwloc 
  $ spack spec -I htop@3.2.0 ~hwloc +debug


Lets go ahead and install htop now. 

.. code-block:: console

  $ spack install htop@3.2.0 ~hwloc +debug
  
To uninstall a spack package. 

.. code-block:: console

  $ spack uninstall libtool@2.4.7

Notice how it fails due to dependencies with packages. 

.. code-block:: console

  ==> Will not uninstall libtool@2.4.7%gcc@7.5.0/mvje3k2
  The following packages depend on it:
    -- linux-ubuntu18.04-haswell / gcc@7.5.0 ------------------------
    ha6adqe htop@3.2.0
  ==> Error: There are still dependents.
    use `spack uninstall --dependents` to remove dependents too

Loading up installed modules 

.. code-block:: console

  $ which htop
  /usr/bin/htop
  $ htop --version
  htop 2.1.0 - (C) 2004-2018 Hisham Muhammad
  Released under the GNU GPL.
  
  $ spack load htop
  $ which htop
  /home/ubuntu/spack/opt/spack/linux-ubuntu18.04-skylake_avx512/gcc-7.5.0/htop-3.2.0-zoznzvyv5ilhshf3at4gqnkhajzgdev7/bin/htop
  $ htop --version
  htop 3.2.0

-------------------
Spack Build Caches 
-------------------

The use of a ``binary cache`` can result in software installs up to 20x faster 
for common Spack package installs. This tutorial will explain through the process 
of setting up a source mirror with a binary cache mirrors. Binary caches allow one 
to install pre-compiled binaries to your spack installation path.

Using the binary cache

.. code-block:: console

  $ spack mirror add tutorial /mirror
  $ spack buildcache keys --install --trust
  
  ==> Fetching https://binaries.spack.io/develop/build_cache/_pgp/2C8DD3224EF3573A42BD221FA8E0CA3C1C2ADA2F.pub
  gpg: key A8E0CA3C1C2ADA2F: 7 signatures not checked due to missing keys
  gpg: key A8E0CA3C1C2ADA2F: public key "Spack Project Official Binaries <maintainers@spack.io>" imported
  gpg: Total number processed: 1
  gpg:               imported: 1
  gpg: no ultimately trusted keys found
  gpg: inserting ownertrust of 6
  
  $ spack mirror list

Now lets take a look inside the buidcache 

.. code-block:: console

  $ spack buildcache list --allarch

This is a very new addition to Spack. The options are limited
and so filtering to specific arch is not yet functional. 

Build caches are hit and miss depending on spack versions and installed packaged. 
For example lammps is not listed in the buildcache mirror list. So most of the install
will still take some time.

Some example commands to try. 

.. code-block:: console

  $ spack spec -I intel-mpi
  $ spack install --cache-only intel-mpi

.. code-block:: console

  $ ==> Installing intel-mpi-2019.10.317-3d3xzc5ibrsjtqvgsv7ewvhdf5uw3ffj
    ==> intel-mpi exists in binary cache but with different hash
    ==> Error: No binary for intel-mpi-2019.10.317-3d3xzc5ibrsjtqvgsv7ewvhdf5uw3ffj found when cache-only specified
    ==> Error: Failed to install intel-mpi due to SystemExit: 1
  
Now lets try to install a package that is listed.

.. code-block:: console

  $ spack buildcache list --allarch | grep intel
  $ spack spec -I intel-tbb
  $ spack install --cache-only intel-tbb

.. code-block:: console

  $ ==> Installing intel-tbb-2020.3-rbexoowaqll5pqen452ef2wqho6jlz36
  ==> Fetching https://binaries.spack.io/develop/build_cache/linux-ubuntu18.04-x86_64-gcc-7.5.0-intel-tbb-2020.3
  rbexoowaqll5pqen452ef2wqho6jlz36.spec.json.sig
  gpg: Signature made Thu Sep  8 19:58:45 2022 UTC
  gpg:                using RSA key D2C7EB3F2B05FA86590D293C04001B2E3DB0C723
  gpg: Good signature from "Spack Project Official Binaries <maintainers@spack.io>" [ultimate]
  ==> Fetching https://binaries.spack.io/develop/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/intel-tbb-2020.3/linux-ubuntu18.04-x86_64-gcc-7.5.0-intel
  tbb-2020.3-rbexoowaqll5pqen452ef2wqho6jlz36.spack
  ==> Extracting intel-tbb-2020.3-rbexoowaqll5pqen452ef2wqho6jlz36 from binary cache
  ==> intel-tbb: Successfully installed intel-tbb-2020.3-rbexoowaqll5pqen452ef2wqho6jlz36
  Search: 0.00s.  Fetch: 1.11s.  Install: 0.53s.  Total: 1.64s
  [+] /home/ubuntu/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/intel-tbb-2020.3-rbexoowaqll5pqen452ef2wqho6jlz36
  
To remove the binary cache from your spack environment. 

.. code-block:: console

  $ spack mirror list
  $ spack mirror remove binary_mirror
  $ spack clean
  $ spack clean -b

-----------------
Spack Compilers
-----------------

Spack can install and manage a list of available compilers on the system, detected 
automatically from the user’s ``PATH`` variable. The ``spack compilers`` command 
is an alias for the command ``spack compiler list``.

.. code-block:: console

  $ spack compilers
  
.. code-block:: console

  ==> Available compilers
  -- gcc ubuntu18.04-x86_64 ---------------------------------------
  gcc@7.5.0
  
Let's install a new compiler 

.. code-block:: console

  $ spack install --cache-only gcc@8.4.0
  
.. code-block:: console

  ==> gcc: Successfully installed gcc-8.4.0-kf55dvoi3iuagjkvomjti2lemura7b42
    Stage: 8.83s.  Autoreconf: 0.00s.  Configure: 2.33s.  Build: 1h 26m 41.56s.  Install: 32.20s.  Total: 1h 27m 25.21s
  [+] /home/ubuntu/spack/opt/spack/linux-ubuntu18.04-skylake_avx512/gcc-7.5.0/gcc-8.4.0-kf55dvoi3iuagjkvomjti2lemura7b42

Now let's add the new compiler to our list of available compilers. Using the 
``spack compiler add`` command. This will allow future packages to build 
with gcc@8.4.0 if selected.

.. code-block:: console

  $ spack find -p gcc
  $ spack compiler add $(spack location -i gcc@8.4.0)
  $ spack compilers

.. code-block:: console

  -- linux-ubuntu18.04-skylake_avx512 / gcc@7.5.0 -----------------
  gcc@8.4.0  /home/ubuntu/spack/opt/spack/linux-ubuntu18.04-skylake_avx512/gcc-7.5.0/gcc-8.4.0-kf55dvoi3iuagjkvomjti2lemura7b42
  ==> 1 installed package
  
  ==> Added 1 new compiler to /home/ubuntu/.spack/linux/compilers.yaml
    gcc@8.4.0
  ==> Compilers are defined in the following files:
    /home/ubuntu/.spack/linux/compilers.yaml
    
  ==> Available compiler
  -- gcc ubuntu18.04-x86_64 ---------------------------------------
  gcc@8.4.0  gcc@7.5.0  
  
Let's use the new version of gcc/8.4.0 and install a few packages. 

.. code-block:: console

  $ spack load gcc@8.4.0
  $ spack find --loaded
  $ spack spec -I bzip2
  $ spack spec -I bzip2%gcc@8.4.0
  $ spack install bzip2%gcc@8.4.0
  $ spack find

The end result should result in packages both installed using ``gcc@7.5.0`` 
and ``gcc@8.4.0``.

Installing gcc/8.4.0 did take 1h 27m total as you can see above. I did not use a build
cache. Let's use a build cache and see how long it takes. 

.. code-block:: console

  $ spack unload gcc@8.4.0
  $ spack buildcache list --allarch | grep gcc
  $ spack install --cache-only gcc@8.4.0
  $ spack find
  
.. code-block:: console

  ==> gcc: Successfully installed gcc-8.4.0-tf5qxoqsrla6jzuno5wdcwsn6saeiy2f
  Search: 0.00s.  Fetch: 12.08s.  Install: 11.64s.  Total: 23.72s
  [+] /home/ubuntu/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.4.0-tf5qxoqsrla6jzuno5wdcwsn6saeiy2f
  
  -- linux-ubuntu18.04-skylake_avx512 / gcc@7.5.0 -----------------
  -- linux-ubuntu18.04-skylake_avx512 / gcc@8.4.0 -----------------
  -- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
  
Notice the difference with the installed packaged / compiler version vs non cache.  

==============================
Building Apptainer Containers
==============================

About Containers
-----------------

Containerized software is becoming more prevelant throughout the computing landscape and that includes research computing. Have you ever had an environment that you have spent hours installing and preparing and then needed to turn around and have a colleague need to replicate it, or worse, you need to migrate to an entirely new system? Containers are prefect for this sort of scenario. If you build it once in a container, the file can be brought and shared to any system that runs a container framework and launch it to run software without worrying about the environment on the local machine.

A container functions as effectively an isolated operating system on a node while it is running. Commands and software executed within the container will therefore run using this isolated system. This has many, many applications but for today we will explore how this can be applied to research workloads.

Two common frameworks for containers in research computing are:
* Docker
* Apptainer/Singularity

We will focus on using Apptainer but note that Docker containers are also supported by Apptainer and infact will be the basis of several containers we will be building.


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
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: console

  $ apptainer exec --fakeroot spack-user-ffmpeg.sif ffmpeg -h

Here we see that the ffmpeg package is installed and ready for use withing the container we built.


Building Apptainer containers from scratch
--------------------------------------------

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

Python Environments in Containers
----------------------------------

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


Advanced Topics with Containers
--------------------------------

Beyond the basics of software building here there are several other more complicated uses of containers that are useful to discuss for HPC usage but we do not have the time to explore with a detailed tutorial.

MPI Workloads and Containers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MPI is a common interface for high performance computing allowing software to make use of multiple nodes for single problems by spreading the memory and computing workload over large numbers of CPUs and sets of system memory. Containers can also be used in these instances but it is important to understand the style and version of MPI interfaces used by the HPC system you will be operating on. 

Ideally when constructing your container the type of MPI software in the container should be similar or identical to the one used on the HPC system for best performance. In many cases it is worthwhile to reach out to the system administration team of the HPC system or review their documentation on how best to use containers with MPI on their system.

GPU usage with Containers
^^^^^^^^^^^^^^^^^^^^^^^^^^

GPUs have become an increasingly powerful and common tool to use with research computing. AI and machine learning software are extremely common users of GPUs but other software is beginning to make use of the accelerated capabilities of GPU processing power as well. Containers can also interface with GPUs for their software as well.

Although we did not have an example to show building a GPU container they can be built much the same as above. Depending on the type of GPU you are utilizing you will need to include the CUDA or ROCm libraries in the container for your software to function as well as make an additional flag during the `apptainer exec` or `apptainer shell` commands to import the GPU devices into the container. These can be activated by using the `--nv` or `--rocm` flags respectively depending on the GPU hardware type.


Hardware Architecture Caveats
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Although containers can create portable software environments, when making your software portable via containers it is important to know the limitations of the software built within the container as well. Many times when software is compiled from source the software will look to optimize for the CPU architecture that is available on the current system. When copying the container to another system it may be that the hardware instructions in the compiled code are not supported on the CPU itself. This will often lead to an 'Instruction Error' being reported and the code failing to start.

Depending on how your software is built it may be possible to over-ride the default of build arctitecture to target a more limited processer instruction set to make your compiled code more portable across multiple arcitectures. Review your software build instructions or compiler flags with 'gcc' or other compilers for how to accomplish this.


Multi-stage Builds
^^^^^^^^^^^^^^^^^^^

To reduce sizes of the final containers and break builds up into multiple layers the 'Stage' tag can be used in container build files. Spack uses this by default with one stage being the build process where sources are installed and built and the second stage moves all of the binaries and required libraries to a new clean container and sets up the environment there.

Designing multi-stage containers from scratch involves more time that we are able to put into the tutorial but further details can be found on Apptainer's documentation pages and from reviewing how systems such as Spack build their containers.


