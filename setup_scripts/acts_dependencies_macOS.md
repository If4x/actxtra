Guide to get all dependencies for macOS for ACTS
================================================

This guide is based on [https://codimd.web.cern.ch/s/w-7j8zXm0](https://codimd.web.cern.ch/s/w-7j8zXm0) (CERN SSO might be required).

This guide was written for macOS 13.3.1  (a) but might be incomplete. You could also have a look at [https://github.com/paulgessinger/ci-dependencies/tree/build_cmake](https://github.com/paulgessinger/ci-dependencies/tree/build_cmake).

Install location
----------------

To keep everything clean all dependecies are tried to be installed in `/opt/hep`. For the installation process we generate a separeted folder.
```console
mkdir setup_dependencies && cd setup_dependencies
```

Brew Packages
-------------

```console
brew install cmake
```

The following still need investigation if really needed:
```console
brew install java
sudo ln -sfn /usr/local/opt/openjdk/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk.jdk
brew install glfw3 glew
```

xerces-c
--------
Does it also work with the current xerces version?
```console
mkdir xerces && cd xerces
wget https://archive.apache.org/dist/xerces/c/3/sources/xerces-c-3.2.4.tar.gz
mkdir source
tar -zxvf xerces-c-3.2.4.tar.gz --strip-components=1 -C source
cmake -S source -B build \
    -G "Unix Makefiles" \
    -DCMAKE_INSTALL_PREFIX=/opt/hep/xerces-c \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_PREFIX_PATH=/usr/local/opt/icu4c
sudo cmake --build build --target install -j8
cd ..
```

boost
-----

```console
mkdir boost && cd boost
wget https://boostorg.jfrog.io/artifactory/main/release/1.82.0/source/boost_1_82_0.tar.gz
mkdir source
tar -zxvf boost_1_82_0.tar.gz --strip-components=1 -C source
cd source
./bootstrap.sh --prefix=/opt/hep/boost
sudo ./b2 install --prefix=/opt/hep/boost
cd ../..
```

eigen
-----

```console
mkdir eigen && cd eigen
git clone https://gitlab.com/libeigen/eigen.git source
cd source
git fetch --tags
git checkout tags/3.4.0
cd ..
cmake -S source -B build \
    -DCMAKE_INSTALL_PREFIX=/opt/hep/eigen \
    -DCMAKE_CXX_STANDARD=17
sudo cmake --build build --target install -j8
cd ..
```

Geant4
------

```console
mkdir geant4 && cd geant4
git clone https://gitlab.cern.ch/geant4/geant4.git source
cd source
git fetch --tags
git checkout tags/v11.1.1
cd ..
cmake -S source -B build \
    -DCMAKE_PREFIX_PATH="/opt/hep/xerces-c" \
    -DCMAKE_INSTALL_PREFIX=/opt/hep/geant4 \
    -DCMAKE_CXX_STANDARD=17 \
    -DGEANT4_USE_GDML=On \
    -DGEANT4_INSTALL_DATA=On \
    -DCMAKE_BUILD_TYPE=Release \
    -DGEANT4_BUILD_TLS_MODEL=global-dynamic \
    -DGEANT4_USE_SYSTEM_EXPAT=ON \
    -DGEANT4_USE_SYSTEM_ZLIB=ON
sudo cmake --build build --target install -j8
cd ..
```

HepMC3
------

```console
mkdir hepmc3 && cd hepmc3
git clone https://gitlab.cern.ch/hepmc/HepMC3.git source
cd source
git fetch --tags 
git checkout tags/3.2.5
cd ..
cmake -S source -B build \
    -DCMAKE_PREFIX_PATH="/opt/hep/xerces-c;/opt/hep/geant4" \
    -DCMAKE_CXX_STANDARD=17 \
    -DCMAKE_INSTALL_PREFIX=/opt/hep/hepmc3 \
    -DCMAKE_BUILD_TYPE=Release \
    -DHEPMC3_BUILD_STATIC_LIBS=OFF \
    -DHEPMC3_ENABLE_PYTHON=OFF \
    -DHEPMC3_ENABLE_ROOTIO=OFF \
    -DHEPMC3_ENABLE_SEARCH=OFF
sudo cmake --build build --target install -j8
cd ..
```

Pythia
------

```console
mkdir pythia && cd pythia
wget https://pythia.org/download/pythia83/pythia8309.tgz
mkdir source
tar -zxvf pythia8309.tgz --strip-components=1 -C source
cd source
./configure --prefix=/opt/hep/pythia8
sudo make install -j8
cd ../..
```

Json
----

```console
mkdir json && cd json
wget https://github.com/nlohmann/json/archive/refs/tags/v3.11.2.tar.gz
mkdir source
tar -zxvf v3.11.2.tar.gz --strip-components=1 -C source
cmake -S source -B build \
    -DCMAKE_INSTALL_PREFIX=/opt/hep/json \
    -DJSON_BuildTests=OFF
sudo cmake --build build --target install -j8
cd ..
```

Root
----
This is a bit troublesome. Apparently there has some problem with this macOS-version and root [https://root-forum.cern.ch/t/building-failed-after-upgrade-to-mac-os-13-3-1/54420/7](https://root-forum.cern.ch/t/building-failed-after-upgrade-to-mac-os-13-3-1/54420/7).

```console
mkdir root && cd root
git clone https://github.com/root-project/root.git source
cd source
git fetch --tags
git checkout tags/v6-28-00-patches
cd ..
cmake -S source -B build \
    -DCMAKE_CXX_STANDARD=17 \
    -DCMAKE_INSTALL_PREFIX="/opt/hep/root" \
    -DCMAKE_PREFIX_PATH="/opt/hep/xerces-c;/opt/hep/geant4;/opt/hep/pythia8;/opt/hep/json" \
    -DCMAKE_BUILD_TYPE=Release \
    -Dfail-on-missing=ON \
    -Dgdml=ON \
    -Dx11=ON \
    -Dpyroot=ON \
    -Ddataframe=ON \
    -Dmysql=OFF \
    -Doracle=OFF \
    -Dpgsql=OFF \
    -Dsqlite=OFF \
    -Dpythia6=OFF \
    -Dpythia8=OFF \
    -Dfftw3=OFF \
    -Dbuiltin_cfitsio=ON \
    -Dbuiltin_xxhash=ON \
    -Dbuiltin_afterimage=ON \
    -Dbuiltin_openssl=ON \
    -Dbuiltin_ftgl=ON \
    -Dgfal=OFF \
    -Ddavix=OFF \
    -Dbuiltin_vdt=ON \
    -Dxrootd=OFF \
    -Dtmva=OFF \
    -Dbuiltin_pcre=ON \
    -Dbuiltin_gsl=ON \
    -Dbuiltin_glew=On \
    -Dbuiltin_gl2ps=On
sudo cmake --build build --target install -j8
cd ..
```

PODIO
-----

```console
mkdir podio && cd podio
git clone https://github.com/AIDASoft/podio.git source
cd source
git fetch --tags
git checkout tags/v00-16-03
cd ..
cmake -S source -B build \
    -DCMAKE_CXX_STANDARD=17 \
    -DCMAKE_INSTALL_PREFIX="/opt/hep/podio" \
    -DCMAKE_PREFIX_PATH="/opt/hep/xerces-c;/opt/hep/geant4;/opt/hep/pythia8;/opt/hep/json;/opt/hep/hepmc3;/opt/hep/root;" \
    -DUSE_EXTERNAL_CATCH2=Off \
    -DBUILD_TESTING=Off
sudo cmake --build build --target install -j8
cd ..

```

EDM4Hep
-------
If you get a `Jinja2`-related error, you could try to use a more recent version of `EDM4Hep`.

It might be required to install these two libraries. Remember to use a virtual enviroment.
```console
pip install jinja2 pyyaml
```

We are not the most recent version `tags/v00-07-02` (at the time of writing), because it does not compile. You could use [patch](https://patch-diff.githubusercontent.com/raw/key4hep/EDM4hep/pull/201.patch) to make it work.

```console
mkdir edm4hep && cd edm4hep
git clone https://github.com/key4hep/EDM4hep.git source
cd source
git fetch --tags
git checkout tags/v00-07
cd ..
cmake -S source -B build \
    -DCMAKE_CXX_STANDARD=17 \
    -DCMAKE_INSTALL_PREFIX="/opt/hep/edm4hep" \
    -DCMAKE_PREFIX_PATH="/opt/hep/xerces-c;/opt/hep/geant4;/opt/hep/pythia8;/opt/hep/json;/opt/hep/hepmc3;/opt/hep/root;/opt/hep/podio" \
    -DUSE_EXTERNAL_CATCH2=Off \
    -DBUILD_TESTING=OFF \
    -DCMAKE_BUILD_TYPE=Release
sudo cmake --build build --target install -j8
cd ..
```

DD4hep
------

```console
export LD_LIBRARY_PATH="/opt/hep/geant4/lib"
source /opt/hep/root/bin/thisroot.sh

mkdir dd4hep && cd dd4hep
git clone https://github.com/AIDASoft/DD4hep.git source
cd source
git fetch --tags
git checkout tags/v01-25-01
cd ..
cmake -S source -B build \
    -DCMAKE_CXX_STANDARD=17 \
    -DCMAKE_INSTALL_PREFIX="/opt/hep/dd4hep" \
    -DDD4HEP_USE_GEANT4=On \
    -DDD4HEP_USE_EDM4HEP=On \
    -DBUILD_TESTING=Off \
    -DCMAKE_BUILD_TYPE=Release \
    -DDD4HEP_USE_XERCESC=ON \
    -DBUILD_DOCS=OFF \
    -DCMAKE_PREFIX_PATH="/opt/hep/xerces-c;/opt/hep/geant4;/opt/hep/pythia8;/opt/hep/json;/opt/hep/hepmc3;/opt/hep/root;/opt/hep/podio;/opt/hep/edm4hep"
sudo cmake --build build --target install -j8
cd ..
```

Other Dependencies (WIP)
========================
THIS DOES NOT WORK
For pyg4ometry I encountered som problems on my machine (macOS 13.3.1). Executing `pip install pyg4ometry` resulted in many error messages similar to `ld: warning: dylib (/usr/local/lib/libTKernel.dylib) was built for newer macOS version (13.0) than being linked (10.9)` after installing all dependencies with brew (cgal, opencascade, cmake, python3X). The problem seems to be with tcl/tk which is a dependecy of opencascade. To solve this problem, we install build it manually.

```console
mkdir tcl && cd tcl
wget http://prdownloads.sourceforge.net/tcl/tcl8.7a5-src.tar.gz
mkdir source
tar -zxvf tcl8.7a5-src.tar.gz --strip-components=1 -C source
cd source/unix
./configure --prefix=/opt/deps/tcl --enable-64bit --enable-framework
make
sudo make install -j8
cd ../../..

mkdir tk && cd tk
wget http://prdownloads.sourceforge.net/tcl/tk8.7a5-src.tar.gz
mkdir source
tar -zxvf tk8.7a5-src.tar.gz --strip-components=1 -C source
cd source/unix
./configure --prefix=/opt/deps/tk --enable-64bit --enable-framework --with-tcl=../../../tcl/source/unix
make
sudo make install -j8
cd ../../..
```


```console
mkdir tcl8613 && cd tcl8613
wget http://prdownloads.sourceforge.net/tcl/tcl8.6.13-src.tar.gz
mkdir source
tar -zxvf tcl8.6.13-src.tar.gz --strip-components=1 -C source
cd source/unix
./configure --prefix=/opt/deps2/tcl --enable-64bit --enable-framework
make
sudo make install -j8
cd ../../..

mkdir tk8613 && cd tk8613
wget http://prdownloads.sourceforge.net/tcl/tk8.6.13-src.tar.gz
mkdir source
tar -zxvf tk8.6.13-src.tar.gz --strip-components=1 -C source
cd source/unix
./configure --prefix=/opt/deps2/tk --enable-64bit --enable-framework --with-tcl=../../../tcl8613/source/unix
make
sudo make install -j8
cd ../../..
```

cmake -S source -B build \
    -DCMAKE_INSTALL_PREFIX=/opt/hep/json \
    -DJSON_BuildTests=OFF
sudo cmake --build build --target install -j8
cd ..
```