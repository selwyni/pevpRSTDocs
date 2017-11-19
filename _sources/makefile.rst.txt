

==============
Makefile
==============

Updated Makefile for MASSIF as of October 17, 2017. ::


  #executable base names
  #serial = evpfft
  mpiparallel = pevpmpifft
  #ompparallel = pvpompfft
  #parallelhist = hpevpmpifft
  # ***************************************************************
  #
  #              System configuration specification
  #
  # ***************************************************************

  help:
  	@echo
  	@echo "  Usage: make [program]"
  	@echo
  	@echo "  Available programs:"
  	@echo "    "$(serial)
  	@echo "    "$(serial)"-debug"
  	@echo "    "$(mpiparallel)
  	@echo "    "$(mpiparallel)"-debug"
  	@echo "    "$(mpiparallel)"-hdf"


  # define the path of the required packages( MPI, FFTW, HDF)

  MPI=/Users/Shared/openmpi-1.8.1
  HDF=/Users/Shared/phdf
  FFTW3=/Users/Shared/fftw-3.3.4
  HDFlibs= ${HDF}/lib -lhdf5 -lhdf5_hl -lhdf5_fortran -lhdf5hl_fortran
  FFTWlibs=${FFTW3}/lib -lfftw3_mpi -lfftw3

  # define the fortran compilers

  FC = gfortran
  PFC = mpif90

  # define the path of include and libraries directory

  INCLUDEDIRS = -I${HDF}/include -I${MPI}/include -I/usr/include -I/usr/local/include -I${FFTW3}/include
  LDFLAGS = -m64 -L${HDF}/lib -lfftw3
  PLDFLAGS = -m64 -L${HDFlibs} -L${FFTWlibs} -lm
  SCLDFLAGS = -m64 -L${FFTWlibs}
  FFLAGS = -m64 -O3 -xf95-cpp-input -fno-second-underscore
  DEBUGFLAGS = -DDEBUG -gdwarf-2 -fbounds-check
  BLAS = -framework veclib -lz -ldl


  count := x
  zero := x
  one := x x

  increment = x $1
  count := $(call increment,$(count))

  #all builds
  MPIFLAGS = ${FFLAGS} -DMPIVERSION -DMPIFFT
  OMPFLAGS = ${FFLAGS} -DOMPVERSION -DOMPFFT
  SCFLAGS = ${FFLAGS} -DMPIVERSION -DSUGARCUBE -DMPI #-fast -ftrapuv -CB -CU -traceback
  P3DFLAGS = ${FFLAGS} -DMPIVERSON -DP3D

  VPATH = ./include:./include/modules:./include/paraFFT

  # dependencies.  Note: order is important.
  modules = \
  global_variables.f90 \
  local_variables.f90 \
  orientations.f90 \
  utility.f90 \
  file_io.f90 \
  cmdargs.f90 \
  viscoplastic.f90 \
  voronoi.f90 \
  random_texture.f90 \
  list_texture.f90 \
  setup.f90 \
  sugarcube_routines.f90 \
  outer_loop.f90 \
  inner_loop.f90



  all : $(serial) $(serial)-debug $(mpiparallel) $(mpiparallel)-debug \
  	$(mpiparallel)-hdf $(sugarcubeparallel) $(sugarcubeparallel)-debug splitup $(parallelhist)

  ## Compile the serial version

  $(serial) : $(modules) evpfft.f90
  ifeq ($(strip $(count)),$(strip $(zero)))
  	@echo "make : No system configuration specified in Makefile"
  	@exit 2
  endif
  ifneq ($(strip $(count)),$(strip $(one)))
  	@echo "make : More than one system configuration specified in Makefile"
  	@exit 2
  endif
  	@echo -n "   Compiling "$@" for "$(BUILD)"..."
  	@$(FC) $(FFLAGS) -o $@ $^ $(INCLUDEDIRS) $(LDFLAGS)
  	@rm -f *.mod
  	@echo "done."

  ## Compile the serial-debug version

  $(serial)-debug: $(modules) evpfft.f90
  ifeq ($(strip $(count)),$(strip $(zero)))
  	@echo "make : No system configuration specified in Makefile"
  	@exit 2
  endif
  ifneq ($(strip $(count)),$(strip $(one)))
  	@echo "make : More than one system configuration specified in Makefile"
  	@exit 2
  endif
  	@echo -n "   Compiling "$@" for "$(BUILD)"..."
  	@$(FC) $(FFLAGS) $(DEBUGFLAGS) -o $@ $^ $(INCLUDEDIRS) $(LDFLAGS)
  	@rm -f *.mod
  	@echo "done."

  ## Compile the parallel version

  $(mpiparallel) : $(modules) evpfft.f90
  ifeq ($(strip $(count)),$(strip $(zero)))
  	@echo "make : No system configuration specified in Makefile"
  	@exit 2
  endif
  ifneq ($(strip $(count)),$(strip $(one)))
  	@echo "make : More than one system configuration specified in Makefile"
  	@exit 2
  endif
  	@echo -n "   Compiling "$@" for "$(BUILD)"..."
  	@$(PFC) $(MPIFLAGS) -o $@ $^ $(INCLUDEDIRS) $(PLDFLAGS)
  	@rm -f *.mod
  	@echo "done."

  ## Compile the parallel-debug varsion

  $(mpiparallel)-debug: $(modules) evpfft.f90
  ifeq ($(strip $(count)),$(strip $(zero)))
  	@echo "make : No system configuration specified in Makefile"
  	@exit 2
  endif
  ifneq ($(strip $(count)),$(strip $(one)))
  	@echo "make : More than one system configuration specified in Makefile"
  	@exit 2
  endif
  	@echo -n "   Compiling "$@" for "$(BUILD)"..."
  	@$(PFC) $(MPIFLAGS) $(DEBUGFLAGS) -DHDF -o $@ $^ $(INCLUDEDIRS) $(PLDFLAGS)
  	@rm -f *.mod
  	@echo "done."

  ## Compile the parallel version conatining HDF libraries

  $(mpiparallel)-hdf: $(modules) evpfft.f90
  ifeq ($(strip $(count)),$(strip $(zero)))
  	@echo "make : No system configuration specified in Makefile"
  	@exit 2
  endif
  ifneq ($(strip $(count)),$(strip $(one)))
  	@echo "make : More than one system configuration specified in Makefile"
  	@exit 2
  endif
  	@echo -n "   Compiling "$@" for "$(BUILD)"..."
  	@$(PFC) $(MPIFLAGS) -DHDF -o $@ $^ $(INCLUDEDIRS) $(PLDFLAGS)
  	@rm -f *.mod
  	@echo "done."




  #EJL UDOT HISTORY
  $(parallelhist) : $(modules) evpfft.f90
  ifeq ($(strip $(count)),$(strip $(zero)))
  	@echo "make : No system configuration specified in Makefile"
  	@exit 2
  endif
  ifneq ($(strip $(count)),$(strip $(one)))
  	@echo "make : More than one system configuration specified in Makefile"
  	@exit 2
  endif
  	@echo -n "   Compiling "$@" for "$(BUILD)"..."
  	@$(PFC) $(MPIFLAGS) -DHIST -o $@ $^ $(INCLUDEDIRS) $(PLDFLAGS)
  	@rm -f *.mod
  	@echo "done."




  clean:
  	@rm -f *.mod
  #	@rm -f $(serial) $(serial)-debug
  #	@rm -f $(mpiparallel) $(mpiparallel)-debug
  #	@rm -f $(ompparallel) $(ompparallel)-debug
  #	@rm -f $(sugarcubeparallel) $(sugarcubeparallel)-debug
  #	@rm -f splitup
