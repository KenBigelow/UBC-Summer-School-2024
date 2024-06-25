
==========================================
Advanced Topics with Containers
==========================================

Beyond the basics of software building here there are several other more complicated uses of containers that are useful to discuss for HPC usage. These topics do require a basic understanding of containers and their build process that we covered in the morning session. 


Python Environments in Containers
===================================

On HPC systems it is common to build virtual environments for python workflows that include several packages. Rebuilding these environments to be the same on multiple systems can be challenging as well as time consuming. Containers can help alleviate this work by building the environment once and making it portable within a single container file.

For our example we will build a container using the 'pandas_environment.txt' file that contains a list of all of the python packages for a conda virtual environment to use the Pandas data analysis library. This example can be extended to any other conda environment as well by exporting or building a requirements file and performing a similar operation on building the container.

To start off we want to work with a container that has the conda software already installed. To do this, rather than starting from a blank Ubuntu image we can actually use a prebuilt image that has miniconda3 already set up.

.. code-block:: singularity

  Bootstrap: docker
  From: continuumio/miniconda3

We next want to bring in our environment file so it is avaiable during the build process so it can be used as a lookup for what packages conda will look to install. The %files tag will copy in any file specified from the local system into the build process so that it can be added to a final image. This is a good way to import source code for self-compiled research work as well.

.. code-block:: singularity

  %files
      pandas_environment.txt

Finally we will want to set up the environment and call the actual build process for the conda installer. Our %environment information is used to ensure that when we launch the container after it is built we have the virtual environement inside already loaded and ready to make python calls against. This requires a bit of extra setup in our %post section to ensure that it is easy for conda to activate the new environment we created.

.. code-block:: singularity

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

.. code-block:: bash

  mpiexec apptainer exec my_container.sif mpi-software --option 1 --setting 2 --input datafile.in



Building and Running MPI Containers
------------------------------------

Software within a container that wishes to interact with MPI will need to be compiled against a matching MPI architecture (eg. OpenMPI or MPICH). In the example we will be exploring today a container has already been placed within the shared storage that was built with OpenMPI bindings. For further reading on how the container is built from a definition file please see: https://apptainer.org/docs/user/1.0/mpi.html#open-mpi-hybrid-container

In the project directory there is a SLURM job script that utilizes the container and executes a basic 'Hello World' program across multiple nodes. Copying this file to your home directory and submitting it will run the software and produce an output file with the list of each node name and rank on that node.

.. code-block:: slurm

  #!/bin/bash
  #SBATCH --job-name apptainer-mpi
  #SBATCH --nodes=2 # total number of nodes
  #SBATCH --tasks-per-node=3
  #SBATCH --account=def-sponsor00
  #SBATCH --time=00:05:00 # Max execution time

  module load gcc openmpi apptainer 
  mpirun apptainer exec /project/mpi_test.sif /opt/mpitest


GPU usage with Containers
===========================

GPUs have become an increasingly powerful and common tool to use with research computing. AI and machine learning software are extremely common users of GPUs but other software is beginning to make use of the accelerated capabilities of GPU processing power as well. Containers can also interface with GPUs for their software as well.

Although we did not have the time to show building a GPU container they can be built much the same as before. Depending on the type of GPU you are utilizing you will need to include the CUDA or ROCm libraries in the container for your software to function as well as make an additional flag during the `apptainer exec` or `apptainer shell` commands to import the GPU devices into the container. These can be activated by using the `--nv` or `--rocm` flags respectively depending on the GPU hardware type.

For our example we will use the latest tensorflow in a container and list all local GPUs. Downloading this container will take a large amount of time so to expedite the example we have already downloaded the container into the course project directory:

.. code-block:: bash
  
  cp /project/tensorflow_list.py .
  cp /project/gpucontainer_job.sh .
  sbatch gpucontainer_job.sh

This will launch the job with the following job script:
  
.. code-block:: slurm

  #!/bin/bash
  #SBATCH --job-name apptainer-mpi
  #SBATCH --nodes=2 # total number of nodes
  #SBATCH --tasks-per-node=3
  #SBATCH --account=def-sponsor00
  #SBATCH --time=00:05:00 # Max execution time
  
  module load gcc apptainer
  apptainer exec --nv /project/tensorflow-latest-gpu.sif python3 tensorflow_list.py
  
  
Multi-stage Builds
===================

To reduce sizes of the final containers and break builds up into multiple layers the 'Stage' tag can be used in container build files. Spack uses this by default with one stage being the build process where sources are installed and built and the second stage moves all of the binaries and required libraries to a new clean container and sets up the environment there.

In the header for a container there is the option to provide the `Stage:` tag to define and break up a single container build into multiple sections of building. This creates a layered approach where early stages can be used to build software dependencies that need substantial additional packages that are not needed in the final output image.

.. code-block:: singularity

  Bootstrap: docker
  From: continuumio/miniconda3
  Stage: build_stage
  
  
Hardware Architecture Caveats
===============================

Although containers can create portable software environments, when making your software portable via containers it is important to know the limitations of the software built within the container as well. Many times when software is compiled from source the software will look to optimize for the CPU architecture that is available on the current system. When copying the container to another system it may be that the hardware instructions in the compiled code are not supported on the CPU itself. This will often lead to an 'Instruction Error' being reported and the code failing to start.

Depending on how your software is built it may be possible to over-ride the default of build arctitecture to target a more limited processer instruction set to make your compiled code more portable across multiple arcitectures. Review your software build instructions or compiler flags with 'gcc' or other compilers for how to accomplish this.

