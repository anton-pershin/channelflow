# Documentation

## Installing Channelflow

Channelflow is hosted on [GitHub](https://github.com/epfl-ecps/channelflow), using the distributed  version control system git.
To obtain a copy of the code clone the current repository running

`git clone https://github.com/epfl-ecps/channelflow`


### Prerequisites

To compile Channelflow:
* CMake (version 3.1 or higher)
* C++11 supportive compiler (tested minimum compiler versions: GNU 4.8.1, Intel 16.0, Clang 6.0)
* FFTW (version 3 or higher)
* Eigen3 (version 3 or higher)

To enable parallelization, to use the NetCDF format (easy visualization with Paraview, VisIt ...) and to enable the parallel I/O:
* MPI
* NetCDF (parallel version 4 or higher)
* HDF5(cxx) (format available for backwards compatibility)

To use Channelflow functions from the Python wrapper:
* boost-python


### Compilation
A Makefile to compile and install Channelflow can be generated using the provided CMake build scripts.
CMake allows two build types: "release" and "debug". The build type decides on compiler options and whether assertions
are compiled. Use "release" for optimal performance (e.g. for production runs) and "debug" for optimal diagnostics
(e.g. for code development).
Out of source builds are recommended.

```
mkdir build
cd build
cmake PATH_TO_SOURCE -DCMAKE_BUILD_TYPE=debug (/release) (configuration options)
make -j
make install
```

Channelflow supports, beneath other standard cmake flags, the following options


|Option                   | Values  | Default   | Description                                                       |
|:------------------------|:--------|:----------|:------------------------------------------------------------------|
|`-DCMAKE_INSTALL_PREFIX` | path    | usr/local | Installation path for make install                                |
|`-DUSE_MPI`              | ON/OFF  | ON        | Enable MPI                                                        |
|`-DWITH_SHARED`          | ON/OFF  | ON        | build shared channelflow and nsolver libraries                    |
|`-DWITH_STATIC`          | ON/OFF  | OFF       | build static libraries (also enables linking to static libraries) |
|`-DWITH_PYTHON`          | ON/OFF  | OFF       | build a python wrapper for flowfields, disabled by default because it requires boost-python |
|`-DWITH_HDF5CXX`         |  ON/OFF | OFF       | enable legacy .h5 file format (using HDF5 C++)                    |


A complete installation, with all features enabled, might look like this:

  `cmake PATH_TO_SOURCE -DCMAKE_BUILD_TYPE=release -DCMAKE_INSTALL_PREFIX=$HOME/usr -DWITH_PYTHON=ON -DWITH_HDF5CXX=ON`

### Compilation on ARC3/ARC4

The fork of Channelflow accounting for ARC4 peculiarities is hosted on [GitHub](https://github.com/anton-pershin/channelflow.git)

Before compiling, one needs to load the gnu compiler (there were certain issues found while trying the intel one), cmake building tool and openmpi, fftw and netcdf libraries:
```
module switch intel/19.0.4 gnu/native
module load cmake/3.15.1
module load openmpi/3.1.4
module load fftw/3.3.8
module load netcdf/4.6.3
```

Since no eigen module is available on ARC3/ARC4, one needs to clone this library manually from https://gitlab.com/libeigen/eigen and pass the header path to cmake command via `-DEIGEN3_INCLUDE_DIR`.

Eventually, building commands may look like this (here we disabled HDF5 and enabled MPI):
```
mkdir build
cd build
cmake -DCMAKE_CXX_COMPILER=mpicxx -DEIGEN3_INCLUDE_DIR=/path/to/eigen3/includes ..
make -j
```

Given that new version of Channelflow makes use of NetCDF4, one may want to use nco tools to perform operations on .nc files (e.g., glue separate .nc files in a single one which is useful when a series of files represents a time series). Since there is no nco package on ARC4, one needs to clone nco from the repository and then compile it:
```
git clone https://github.com/nco/nco.git
module switch intel/19.0.4 gnu/native
module load cmake/3.15.1
module load netcdf/4.6.3
module load hdf5/1.8.21
cd nco && mkdir build && cd build && cmake -DNETCDF_INCLUDE=/apps/developers/libraries/netcdf/4.6.3/1/gnu-native-openmpi-3.1.4/include -DNETCDF_LIBRARY=/apps/developers/libraries/netcdf/4.6.3/1/gnu-native-openmpi-3.1.4/lib/libnetcdf.so -DHDF5_LIBRARY=/apps/developers/libraries/hdf5/1.8.21/1/gnu-native-openmpi-3.1.4/lib/libhdf5.so -DHDF5_HL_LIBRARY=/apps/developers/libraries/hdf5/1.8.21/1/gnu-native-openmpi-3.1.4/lib/libhdf5_hl.so ..
make -j
```

### Tests

The correct functionality of the Channelflow code is tested in two ways:
* **Unit tests** are tests of individual classes or methods.
* **Integration tests** are tests of applications with all underlying code.

Both test types are hooked to the `make test` command, where they run sequentially.
If all tests pass successfully, the installation was successful.
When `make test` hangs on executing a test, re-run the tests and see if the problem persists.
For verbose output and debugging, run `make test ARGS="-V"`

Unit tests use the [googletest](https://github.com/google/googletest) framework, which comes bundled with channelflow in its version 1.8.0.
To run only the unit tests, run `tests/gtest/runUnitTest` in the CMake build directory.
Individual tests can be run by passing `--gtest_filter=<foo>` as an argument, where `<foo>` is a name or pattern of a specific unit test.
One example would be `tests/gtest/runUnitTest --gtest_filter=TimeStepTest.*` to run all tests of the time stepping class.
More on unit testing can be found in the [googletest Documentation](https://github.com/google/googletest/blob/master/README.md).

Integration tests are found in folder `tests`, and registered in `tests/CMakeList.txt`. They are compiled into standalone
programs, which are individually listed in `tests/CMakeList.txt` from where they are called on `make test`.
For debugging, each integration test can be run individually and with various command-line arguments.

### Doxygen

A webpage based on [Doxygen](http://www.doxygen.nl) is generated by running `make doc` (if doxygen is installed).
A directory named `html` is automatically generated in the build directory. To access the webpage, open the file `html/index.html`.

### Running your first simulations

To run a DNS we first need to provide an initial velocity field. This can be done providing a specific file or creating
a random flowfield using the utility `randomfield` in the `tools` folder.

Running the command line

`randomfield -Nx 48 -Ny 81 -Nz 32 -Lx 3 -Lz 2 newfield.nc`

a random flowfield with zero-divergence, Dirichlet boundary conditions and the specified resolution and geometry is created.

Time integration of this initial condition is performed by the program `simulateflow` in the `programs` folder.
The specific system to be simulated is specified via the program's options.

The following example integrates for 200 advective time units a simple plane Couette system at the default Reynolds number
400 .

`simulateflow -T 200 newfield.nc`

The following example simulates a pressure driven (-dPds) Poiseuille flow (-Uwall 0) at the Reynolds number 1000 using
a 1st order forward-backward euler scheme as initial time stepping algorithm (-is).

`simulateflow -R 1000 -Uwall 0 -dPds -0.002 -is SBDF1 newfield.nc`

### Running simulations on ARC3/ARC4

Below is an example of the script for qsub running a time-integration employing 24 cores for maximum 12 hours (note `mpirun` before the command):
```
#$ -cwd -V
#$ -l h_rt=12:00:00
#$ -pe ib 24
mpirun /path/to/channelflow/programs/simulateflow -T0 0 -T 3000 -dT 0.5 -s 10 -R 250 -dt 0.004 -o data initial_condition.nc
```
