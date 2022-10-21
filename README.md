[![Test HDF5 GPUDirect Storage VFD](https://github.com/hyoklee/vfd-gds/actions/workflows/main.yml/badge.svg)](https://github.com/hyoklee/vfd-gds/actions/workflows/main.yml)
[![Install GDS/CUDA via Yum](https://github.com/hyoklee/vfd-gds/actions/workflows/yum.yml/badge.svg)](https://github.com/hyoklee/vfd-gds/actions/workflows/yum.yml)
[![Test CUDA Ubuntu-2004](https://github.com/hyoklee/vfd-gds/actions/workflows/cmake.yml/badge.svg)](https://github.com/hyoklee/vfd-gds/actions/workflows/cmake.yml)

# HDF5 Nvidia GPUDirect Storage VFD

## Table of Contents
1. [Description](#1-Description)
2. [Installation](#2-Installation)
    * [Prerequisites](#Prerequisites)
    * [Build instructions](#Build-instructions)
        * [CMake](#CMake-build-instructions)
            * [CMake options](#CMake-build-options)
            * [Driver options](#CMake-driver-options)
3. [Usage and Testing](#3-Usage-And-Testing)
    * [Linked into application](#Linked-into-application)
    * [Dynamically loaded by FAPL](#Dynamic-load-fapl)
    * [Dynamically loaded by environment variable](#Dynamic-load-variable)
    * [Usage](#Driver-Usage)
4. [Known Issues and Limitations](#4-Known-Issues-And-Limitations)
5. [More Information](#5-More-Information)

## 1. Description

The HDF5 GPUDirect Storage VFD is a Virtual File Driver (VFD) for HDF5 that can
be used to interface with Nvidia's GPUDirect Storage (GDS) API. The driver is
built as a plugin library that is external to HDF5.

## 2. Installation

### Prerequisites

To build the GDS VFD, the following libraries are required:

+ `libhdf5` - The [HDF5](https://www.hdfgroup.org/downloads/hdf5/) library.
            Minimum version required is 1.13.0. HDF5 must be compiled with
            support for parallel I/O if building the GDS VFD's tests. Shared
            libraries must be enabled if loading the GDS VFD as an HDF5 plugin.

+ `libcuda` - Nvidia [CUDA](https://developer.nvidia.com/about-cuda) library.
            Cufile support must also be available.

+ `libmpi` - MPI library. The GDS VFD's tests use MPI to test functionality
            of the driver.

Compiled libraries must either exist in the system's library paths or must be
pointed to during the GDS VFD build process.

### Build instructions

The HDF5 GDS VFD is built using CMake.

#### CMake

CMake version 2.8.12.2 or greater is required for building the driver.

After obtaining the driver's source code, you can create a build directory
within the source tree and run the `ccmake` or `cmake` command from it:

    cd vfd-gds-X
    mkdir build
    cd build
    ccmake ..

If using `ccmake`, type `'c'` multiple times and choose suitable options or if
using `cmake`, pass these options with `-D`. Some of these options may be needed
if, for example, the required components mentioned previously are not located in
default paths.

Setting include directory and library paths may require you to toggle to
the advanced mode by typing `'t'`. Once you are done and do not see any
errors, type `'g'` to generate makefiles. Once you exit the CMake
configuration screen and are ready to build the targets, do:

    make

Verbose build output can be generated by appending `VERBOSE=1` to the
`make` command.

Assuming that the `CMAKE_INSTALL_PREFIX` has been set and that you have
write permissions to the destination directory, you can install the driver
by simply doing:

     make install

##### CMake options

  * `CMAKE_INSTALL_PREFIX` - This option controls the install directory that the
  resulting output files are written to. The default value is `/usr/local`.
  * `CMAKE_BUILD_TYPE` - This option controls the type of build used for the VFD.
  Valid values are `Release`, `Debug`, `RelWithDebInfo`, `MinSizeRel`; the default
  build type is `RelWithDebInfo`.

##### Driver options

  * `BUILD_EXAMPLES` - This option is used to enable/disable building of the
  GDS VFD's HDF5 example programs. The default value is `ON`.
  * `BUILD_TESTING` - This option is used to enable/disable building of the
  GDS VFD's tests. The default value is `ON`.
  * `HDF5_VFD_GDS_CUFILE_DIR` - This option controls the directory used when
  searching for the NVIDIA GPUDirect Storage cuFile library. This option may
  need to be set if CMake cannot automatically resolve the path.
  * `HDF5_VFD_GDS_CUFILE_LIB` - This option controls the library used for
  NVIDIA GPUDirect Storage cuFile support. This option may need to be set if
  CMake cannot automatically resolve it.
  * `HDF5_C_COMPILER_EXECUTABLE` - This option controls the HDF5 compiler
  wrapper script used by the VFD build process. It should be set to the full
  path to the HDF5 compiler wrapper (usually `bin/h5cc`), including the name
  of the wrapper script. The following two options may also need to be set.
  * `HDF5_C_LIBRARY_hdf5` - This option controls the HDF5 library used by the
  VFD build process. It should be set to the full path to the HDF5 library,
  including the library's name (e.g., `/path/libhdf5.so`). Used in conjunction
  with the `HDF5_C_INCLUDE_DIR` option.
  * `HDF5_C_INCLUDE_DIR` - This option controls the HDF5 include directory used
  by the VFD build process. Used in conjunction with the `HDF5_C_LIBRARY_hdf5`
  variable.
  * `MPI_C_COMPILER` - This option controls the MPI C compiler used by the VFD
  build process. It should be set to the full path to the MPI C compiler,
  including the name of the executable.

## 3. Usage and Testing

To use the HDF5 GDS VFD in an HDF5 application, the driver can either be linked
into the application, or it can be dynamically loaded as a plugin. If dynamically
loading the GDS VFD, users should ensure that the `HDF5_PLUGIN_PATH` environment
variable points to the directory containing the built VFD library if the VFD has
been installed to a non-standard location.

### Linked into application

To link the GDS VFD into an HDF5 application, the application should include the
`H5FDgds.h` header that gets installed on the system and should link the installed
VFD library (`libhdf5_vfd_gds.so`, or similar) into the application. Once this has
been done, GDS VFD access can be setup by calling `H5Pset_fapl_gds(...)` on a FAPL
within the HDF5 application. Refer to the HDF5 example programs under the `examples`
folder to see how this is done.

### Dynamically loaded by FAPL

To explicitly load the GDS VFD inside an HDF5 application, a call to the
`H5Pset_driver_by_name(...)` routine should be made to setup GDS VFD access on a
FAPL. This will cause HDF5 to load the VFD as a plugin and set the VFD on the
given FAPL. The string "gds" should be given for the `driver_name` parameter. NULL
should be given for the `driver_config` parameter, as the driver does not currently
accept configuration strings.

### Dynamically loaded by environment variable

To implicitly load the GDS VFD inside an HDF5 application, the `HDF5_DRIVER`
environment variable may be set to the string "gds". During library initialization,
HDF5 will check this environment variable, load the GDS VFD as a plugin and set the
VFD as the default file driver on File Access Property Lists. Therefore, any file
access that uses `H5P_DEFAULT` for its FAPL, or which uses a FAPL that hasn't had a
specific VFD set on it, will automatically use the GDS VFD for file access.

### Usage

Users of the GDS VFD should be aware that using the VFD in a normal, unmodified HDF5
application will cause the VFD to behave similarly to the sec2 VFD; that is, the VFD
will make use of POSIX I/O API routines to do file I/O. To make proper use of
GPUDirect storage-capable devices with the VFD, the HDF5 application should make use
of CUDA memory management API routines (e.g., cudaMalloc) to allocate memory on the
device. Then, the device-allocated memory buffer should be passed to `H5Dread`/`H5Dwrite`.
For examples of this, refer to the HDF5 programs under the `examples` folder.

## 4. Known Issues and Limitations

  * For chunked datasets, HDF5 may attempt to cache chunks in its chunk cache
    under certain circumstances, which can incur overhead when copying between
    CUDA device memory and host memory. This caching mechanism cannot currently be
    properly disabled; therefore, performance may suffer for chunked datasets if:
    - filters are applied to the dataset's chunks
    - fill values need to be written to chunks

## 5. More Information

[GPUDirect Storage](https://developer.nvidia.com/blog/gpudirect-storage/)

[HDF5 VFD Plugins RFC](https://github.com/HDFGroup/hdf5doc/blob/master/RFCs/HDF5_Library/VFL_DriverPlugins/RFC__A_Plugin_Interface_for_HDF5_Virtual_File_Drivers.pdf)
