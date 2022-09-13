# Installation instructions

At present, there is no supported standard installation process.

---

## Building and running with docker [TESTED on MacBook with M1Pro]:

The quickest way to build and run NGEN is to follow what we do for our [model run testing](./docker/CENTOS_NGEN_RUN.dockerfile):

First we cd to a directory:
`cd code`

Then clone the github repository:
`git clone git@github.com:NOAA-OWP/ngen.git`

Then we can build with docker:
`docker build . --file ./docker/CENTOS_NGEN_RUN.dockerfile --tag localbuild/ngen:latest`

Finally, we can run the NGEN model on example data:
`docker run --rm -i localbuild/ngen:latest`

Note that, internal command (e.g., start with CMD in the last line of Dockerfile) 
has hardcoded paths to repo-local data When running command manually from container use below:
```
cd ..
./gccbuild/ngen data/catchment_data.geojson "" data/nexus_data.geojson "" data/example_realization_config.json
```

## Building manually on my mac:

**Download the Boost Libraries:**

`
cd /Users/ljzhu/Documents/src
curl -L -O https://boostorg.jfrog.io/artifactory/main/release/1.72.0/source/boost_1_72_0.tar.bz2 \
    && tar -xjf boost_1_72_0.tar.bz2 \
    && rm boost_1_72_0.tar.bz2
`
set BOOST_ROOT="/Users/ljzhu/Documents/src/boost_1_72_0"
So, we can set `BOOST_ROOT` when building with cmake:
`-DBOOST_ROOT=/Users/ljzhu/Documents/src/boost_1_72_0`

**Build NGEN using cmake:**

Using GCC:

+ Use `-DBoost_NO_BOOST_CMAKE=ON` to avoid CMake to look for the 
package configuration file called `BoostConfig.cmake` or `boost-config.cmake` 
and stores the result in `CACHE` entry `Boost_DIR`. 
+ Use pyenv to create a new python virtual environment for NGEN
    ```
    brew install pyenv
    # Install the target version, e.g., 3.8.x specified in docker/CENTOS_NGEN_RUN.dockerfile
    pyenv install 3.8.13
    # use this python version to create virtualenv (recommend to use full python path)
    /Users/ljzhu/.pyenv/versions/3.9.9/bin/python3 -m venv .venv
    # install numpy
    /Users/ljzhu/Documents/code/ngen/.venv/bin/pip3 install numpy
    ```

`
mkdir gccbuild && cd gccbuild
cmake -DCMAKE_C_COMPILER=/opt/homebrew/bin/gcc-12 -DCMAKE_CXX_COMPILER=/opt/homebrew/bin/g++-12 -DBoost_NO_BOOST_CMAKE=ON -DBOOST_ROOT=/Users/ljzhu/Documents/src/boost_1_72_0 ..
cmake --build . --target ngen
`

**Running the Model:**
`
./ngen data/catchment_data.geojson "" data/nexus_data.geojson "" data/example_realization_config.json
`

## Building manually on Linux:

**Install the Linux required packages (package names shown for Centos 8.4.2105):**

`
yum install tar git gcc-c++ gcc make cmake python38 python38-devel python38-numpy bzip2 udunits2-devel texinfo
`

**Make a directory to contain all the NGEN items:**

`
mkdir ngen && cd ngen
`

**Download the Boost Libraries:**

`
curl -L -O https://boostorg.jfrog.io/artifactory/main/release/1.72.0/source/boost_1_72_0.tar.bz2 \
    && tar -xjf boost_1_72_0.tar.bz2 \
    && rm boost_1_72_0.tar.bz2
`

**Set the ENV for Boost and C compiler:**

`
set BOOST_ROOT="/boost_1_72_0"
`
`
set CXX=/usr/bin/g++
`

**Get the git submodules:**

`
git submodule update --init --recursive -- test/googletest 
`
`
git submodule update --init --recursive -- extern/pybind11
`

**Build NGEN using cmake:**

`
cmake -B /ngen -S . &&
cmake --build . --target ngen
`

**Running the Model:**
`
./ngen data/catchment_data.geojson "" data/nexus_data.geojson "" data/example_realization_config.json
`

---

**Further information on each step:**

Make sure all necessary [dependencies](doc/DEPENDENCIES.md) are installed, and then [build the main ngen target with CMake](doc/BUILDS_AND_CMAKE.md).  Then run the executable, as described [here for basic use](README.md#usage) or [here for distributed execution](doc/DISTRIBUTED_PROCESSING.md#examples).
