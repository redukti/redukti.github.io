====================
Building OpenRedukti
====================

Dependencies
============

OpenRedukti makes use of following external libraries:

* `Protocol Buffers <https://developers.google.com/protocol-buffers/>`_ is used to implement data types
* `OpenBLAS <http://www.openblas.net/>`_ is used for Linear Algebra
* `LAPACK <http://www.netlib.org/lapack/>`_ is used for Linear Algebra
* `CMake <https://cmake.org/>`_ is used to generate build scripts 

Optionally if you want to enable a gRPC based server application then additional dependency on:

* `gRPC <https://grpc.io/>`_ framework

Note that the gRPC server is required if you want to use the pricing and curve building functions from the `Python interface <https://github.com/redukti/PyRedukti>`_.

Build Instructions for RHEL 7.6
===============================
I am using gcc 8.3 on Redhat obtained via devtoolset-8.
I installed CMake manually as follows::

  mkdir ~/Software
  cd ~/Software
  wget -O "cmake-3.14.5-Linux-x86_64.tar.gz" "https://github.com/Kitware/CMake/releases/download/v3.14.5/cmake-3.14.5-Linux-x86_64.tar.gz"
  tar xvf "cmake-3.14.5-Linux-x86_64.tar.gz"
  rm "cmake-3.14.5-Linux-x86_64.tar.gz"
  export PATH=/Software/cmake-3.14.5-Linux-x86_64/bin:${PATH}

I enabled `EPEL <https://fedoraproject.org/wiki/EPEL>`_ repository. This was necessary to obtain openblas which I then installed as follows::

  sudo yum install -y openblas-devel readline-devel

Build protobuf
--------------
I built protobuf as follows::

  mkdir -p ~/sources
  cd ~/sources
  wget "https://github.com/protocolbuffers/protobuf/releases/download/v3.8.0/protobuf-cpp-3.8.0.tar.gz" -O "protobuf-cpp-3.8.0.tar.gz"
  tar xvf "protobuf-cpp-3.8.0.tar.gz"
  cd protobuf-3.8.0
  ./autogen.sh
  ./configure --prefix=~/Software/protobuf
  make install

Build gRPC
----------
I built gRPC as follows::

  mkdir -p ~/sources
  cd ~/sources
  git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc
  cd grpc
  git submodule update --init
  sed -i 's/\-Werror//g' third_party/boringssl/CMakeLists.txt
  sed -i 's/\-Werror//g' Makefile
  prefix=~/Software/grpc make install
	
Build OpenRedukti
-----------------
OpenRedukti was built as follows::

  mkdir -p ~/sources
  cd ~/sources
  git clone https://github.com/redukti/OpenRedukti.git
  cd OpenRedukti
  mkdir cppbuild
  cd cppbuild
  cmake -DGRPC_SERVER=ON -DProtobuf_ROOT=~/Software/protobuf -DCMAKE_INSTALL_PREFIX=~/Software/redukti -DGRPC_ROOT=~/Software/grpc ..
  make install

You can omit the ``GRPC_SERVER`` option if that is not needed.

Running OpenRedukti server
--------------------------
::

  export LD_LIBRARY_PATH=$HOME/Software/protobuf/lib:$HOME/Software/redukti/lib:$HOME/Software/grpc/lib:$LD_LIBRARY_PATH
  $HOME/Software/redukti/bin/reduktiserv -s $HOME/Software/redukti/scripts/pricing.lua -p 9001

Build Instructions for Ubuntu Linux 18.04 LTS
=============================================

Following instructions do not build gRPC server.

Pre-Requisites
--------------

Install following::

    sudo apt install git
    sudo apt install cmake
    sudo apt install libreadline-dev
    sudo apt install libopenblas-dev
    sudo apt install libprotobuf-dev
    sudo apt install protobuf-compiler

Build OpenRedukti
-----------------

Clone the OpenRedukti github repository and do following:: 

    mkdir buildrelease
    cd buildrelease
    cmake -DCMAKE_INSTALL_PREFIX=~/Software/OpenRedukti -DCMAKE_BUILD_TYPE=Release ..
    make install

Build Instructions for Windows
==============================
These are instructions for building OpenRedukti on Windows 10 64-bit.

Setup OpenBLAS and LAPACK
-------------------------
These are available as pre-built packages from `Ravi Distribution Dependencies <https://github.com/dibyendumajumdar/ravi-external-libs>`_. 
We assume here that the installed libraries are under ``c:\Software\OpenRedukti``. 

If you have your OpenBLAS and LAPACK files installed differently, please review and amend the ``FindOpenBLAS.cmake`` file in the ``cmake`` folder.

Build Protocol Buffers
----------------------
I had to build protobuf locally. 
I installed the Protobuf binaries into following locations:

* ``c:\Software\protobuf371r`` - release version
* ``c:\Software\protobuf371d`` - debug version

Build gPRC
----------
This is an optional step. 

On Windows, you can build and install gRPC using `vcpkg`. This is what I did.
Or else follow instructions at `gRPC C++ Building from source <https://github.com/grpc/grpc/blob/master/BUILDING.md>`_.  

Build OpenRedukti without gRPC
------------------------------
Once all of above steps are done, you can build OpenRedukti as follows::

	mkdir build
	cd build
	cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_INSTALL_PREFIX=c:\Software\OpenRedukti -DCMAKE_BUILD_TYPE=Release -DProtobuf_ROOT=c:\Software\protobuf371r ..
	
Building OpenRedukti with gRPC
------------------------------

::

	mkdir build
	cd build
	cmake  -DCMAKE_INSTALL_PREFIX=c:\Software\OpenRedukti -G "Visual Studio 15 2017 Win64" -DCMAKE_BUILD_TYPE=Release -DProtobuf_ROOT=c:\Software\protobuf371r -DgRPC_DIR=c:\work\vcpkg\installed\x64-windows-static-dyncrt\share\grpc -Dc-ares_DIR=c:\work\vcpkg\installed\x64-windows-static-dyncrt\share\c-ares ..

Above creates projects suited for debug build. You can go into VS2017 and do the build from there.

For a release build, do following::

	mkdir buildrelease
	cd buildrelease
	cmake -DCMAKE_INSTALL_PREFIX=c:\Software\OpenRedukti -G "Visual Studio 15 2017 Win64" -DCMAKE_BUILD_TYPE=Release ..

Remember to select Release configuration in VS2017. You can run the INSTALL target to copy the final binaries to the installation location specified with ``-DCMAKE_INSTALL_PREFIX`` option.

