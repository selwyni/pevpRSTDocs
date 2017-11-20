================
Building MASSIF
================

.. toctree::
   :maxdepth: 2
   :caption: Mac OSX
   :hidden:


The implementation of MASSIF at Carnegie Mellon University is capable of taking advantage of MPI for parallel computation and the HDF5 API for compressed hierarchial storage of the simulation output. A new computer that needs MASSIF setup will require the following

  * gcc
  * gfortran
  * OpenMPI
  * FFTW
  * Parallel-enabled HDF5 API

As of November 2017, no bundled installer exists for MASSIF. It is highly recommended that the user build the dependencies from source, as OpenMP and Parallel HDF5 are machine-hardware dependent. The following tutorial will explain how to install the libraries.

-----------
Mac OSX
-----------

Mac OSX has several package managers available. Homebrew is the package manager used in this tutorial, and can be found at `<https://brew.sh>`_.

I encourage that the following steps be done in a separate working directory. This can be done in terminal with ::

  mkdir ~/Desktop/builddir
  cd ~/Desktop/builddir

^^^^^^^^^^^^^^^
gFortran, gcc
^^^^^^^^^^^^^^^

With brew installed, open up your terminal and run the following ::

    brew install gcc

To confirm that this installed correctly, the following should provide some informational output. ::

    gcc --version
    gfortran --version

gcc and gfortran are compilers for C and Fortran code respectively.

^^^^^^^^^^^^^^^
Open-MPI
^^^^^^^^^^^^^^^

OpenMPI can be downloaded at `<https://www.open-mpi.org/software/ompi/>`_. Unpack the tar.gz file in a working directory, and check that the configure file exists. Then run the following commands, inserting your own desired output path in the prefix flag. ::

  cd ~/Desktop/builddir
  ./configure --prefix="/Users/user/Desktop/OMPIPATH"
  make all
  make install

At this point, ensure the OpenMPI has been added to the PATH environment variable, by adding the following line to the ``~/.bashrc`` file. The file may or may not exist at the time. ::

  export PATH="/Users/user/Desktop/OMPIPATH:${PATH}"

Then run ``source ~/.bashrc`` in terminal to add the OMPIPATH to the path variable. Run ::

  which mpicc
  which mpif90

to ensure that the MPI-enabled C and Fortran compilers are available, as well as from the same directory OpenMPI was installed in.

^^^^^^^^^^^^^^
FFTW
^^^^^^^^^^^^^^

FFTW can be downloade from `<https://www.fftw.org/download.html>`_. A version in the 3.3 range is preferable, although older version of MASSIF may run in v2.1.5. Unpack the tar.gz in the working direction, and check that the configure file exists. Then run the following command in terminal with the desired output path in the prefix flag. ::

  cd ~/Desktop/builddir
  ./configure --prefix="/Users/user/Desktop/FFTWPATH/" --enable-mpi
  make
  make install

Check that the output directory stated in the prefix flag contains four directories. Within the ``include`` directory, check that there are FFTW files with ``-mpi`` in the filename.

^^^^^^^^^^^^^^^
Parallel HDF5
^^^^^^^^^^^^^^^

Download the HDF5 API from `<https://support.hdfgroup.org/downloads/index.html>`_ and unpack the tar.gz in the working directory. Then run the following commands and insert your desired output path in the prefix flag. ::

  ./configure CC=mpicc FC=mpif90 --prefix="/Users/user/Desktop/PHDF5PATH" --enable-fortran --enable-parallel
  make
  make install

^^^^^^^^^^^^^^^^^
Compiling MASSIF
^^^^^^^^^^^^^^^^^

Configure your path variable to point towards the directories containing the Parallel HDF5, FFTW, and OpenMPI. Add the following lines to your ``~/.bashrc``, omitting the OpenMPI addition if it exists already. Also ensure that the paths point to directories containing the relevant ``bin``, ``lib`` and ``include`` directories. ::

  export PATH="/Users/user/Desktop/OMPIPATH:${PATH}"
  export PATH="/Users/user/Desktop/FFTWPATH:${PATH}"
  export PATH="/Users/user/Desktop/PHDF5PATH:${PATH}"

Run ``source ~/.bashrc`` to add these.

At this point, go into the MASSIF Makefile and adjust the following flags. Also ensure that the correct Fortran compilers are used. ::

  MPI=/Users/user/Desktop/OMPIPATH
  HDF=/Users/user/Desktop/PHDF5PATH
  FFTW3=/Users/user/Desktop/FFTWPATH

  # Define Fortran Compilers
  FC = gfortran
  PFC = mpif90

Save the makefile, and run ``make help`` to see the available builds of MASSIF.
