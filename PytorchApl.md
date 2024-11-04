# Building for Windows on ARM64

1. [Prerequisites](#prerequisites)
1. [Cloning PyTorch](#cloning-pytorch)
1. [Preparing Environment](#preparing-environment)
1. [Setting Environment Variables](#setting-environment-variables)
1. [Build](#build)
   1. [PyTorch](#pytorch)
   1. [LibTorch](#libtorch)


## Prerequisites

1. [Build Tools for Visual Studio 2022](https://aka.ms/vs/17/release/vs_BuildTools.exe) **OR** any version of [Visual Studio 2022](https://visualstudio.microsoft.com/downloads/)
   
   - Under **Workloads**, select **Desktop development with C++**

   - Under **Individual Components**, select **ARM64/ARM64EC build tools (latest)**

1. [Python 3.12 Windows installer (ARM64)](https://www.python.org/downloads/windows/)

1. Download and install [Arm Performance Libraries for Windows](https://developer.arm.com/Tools%20and%20Software/Arm%20Performance%20Libraries#Downloads)

   - `%ARMPL_DIR%` environment variable should be automatically added to your `PATH` by default installation settings.

1. Install [Rust](https://static.rust-lang.org/rustup/dist/x86_64-pc-windows-msvc/rustup-init.exe)

1. **(OPTIONAL)** You may install [sccache](https://github.com/mozilla/sccache/releases) for speeding up future builds by cache support.

   - You need to add `sccache` to your `PATH` **manually**.

## Cloning PyTorch

1. Clone the PyTorch repository (or your fork)

   ```cmd
   git clone --recursive https://github.com/pytorch/pytorch
   cd pytorch
   ```

1. Init and sync submodules

   ```cmd
   git submodule sync
   git submodule update --init --recursive
   ```

## Preparing Environment

> NOTE: Since currently `conda` [doesn't support Windows ARM64](https://github.com/conda/conda/issues/11472), instead we're recommending to use built-in Python `venv` functionality for similar isolating experience. ([more info](https://docs.python.org/3/library/venv.html))

1. Creating a virtual environment (`venv`)

   ```cmd
   python -m pip install --upgrade pip
   python -m venv .venv
   echo * > .venv\.gitignore
   call .\.venv\Scripts\activate
   ```

1. Install requirements

   ```cmd
   pip install -r requirements.txt
   ```

1. Activate `Arm64 Native Tools Command Prompt for VS 2022` environment by calling one of the following commands based on what you've installed.

   - For Build Tools:

      > "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvarsall.bat" arm64

   - For Visual Studio 2022 Community:

      > "C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvarsall.bat" arm64

   - For Visual Studio 2022 Enterprise:
   
      > "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\VC\Auxiliary\Build\vcvarsall.bat" arm64

   - For Visual Studio 2022 Preview:
    
      > "C:\Program Files\Microsoft Visual Studio\2022\Preview\VC\Auxiliary\Build\vcvarsarm64.bat" arm64
   
   

## Setting Environment Variables
1. Set the following environment variables:

   ```cmd
   set BLAS=APL
   set USE_LAPACK=1
   set CMAKE_C_COMPILER_LAUNCHER=sccache
   set CMAKE_CXX_COMPILER_LAUNCHER=sccache
   set MSSdk=1
   set DISTUTILS_USE_SDK=1
   ```

1. **(OPTIONAL)** If you want to use `sccache` for speeding up your future build by cache also set following.

   ```cmd
   set CMAKE_C_COMPILER_LAUNCHER=sccache
   set CMAKE_CXX_COMPILER_LAUNCHER=sccache
   ```

## Build

You can generate a `.whl` (wheel) file for **PyTorch** (the Python package) **OR** a `.zip` file for **LibTorch** (the C++ core). If you'd like to use PyTorch right away without creating a wheel file, you can build it directly in your environment. Here are your options:

### PyTorch

1. **Generate a wheel file** (for distribution or installation on other systems):

   Call following command to start build and generate output
   ```
   :: set PYTORCH_BUILD_VERSION=2.6.0   ## OPTIONAL: If you wish to give a specific versioning for `.whl` file 
   :: set PYTORCH_BUILD_NUMBER=1        ## (for instance you can specify 2.6.0 as shown) 
   python setup.py bdist_wheel
   ```

   Output file will be under `dist` folder with `.whl` extension.
   
   Format will look like `torch-FOO+gitBAR-cp312-cp312-win_arm64.whl`
**OR** if you specify version, format will be like `torch-2.6.0-cp312-cp312-win_arm64.whl`

   Now you can copy and install this `.whl` file anywhere in your system by calling

   ```
   pip install torch-YOURVERSION-cp312-cp312-win_arm64.whl
   ```

1. **Without generating wheel file** (for development on your system only)

   Run one of the following commands to initiate the build:

   - Install the package in development mode, ideal when making frequent changes to codebase
   ```
   python setup.py develop 
   ```

   - Install the package for regular use, rather than ongoing development
   ```
   python setup.py install 
   ```

### LibTorch

1. Additional preparation for `libtorch` folder

   ```cmd
   :: Options: Release, Debug or RelWithDebInfo
   set CMAKE_BUILD_TYPE=Release

   :: Prepare the environment
   mkdir libtorch
   mkdir libtorch\bin
   mkdir libtorch\cmake
   mkdir libtorch\include
   mkdir libtorch\lib
   mkdir libtorch\share
   mkdir libtorch\test
   ```

1. Call following command to start build and generate output

   ```cmd
   python ./tools/build_libtorch.py
   ```

1. Prepare `.zip` file from `libtorch` output

   ```cmd
   :: Moving the files to the correct location
   move /Y torch\bin\*.* libtorch\bin\
   move /Y torch\cmake\*.* libtorch\cmake\
   robocopy /move /e torch\include\ libtorch\include\
   move /Y torch\lib\*.* libtorch\lib\
   robocopy /move /e torch\share\ libtorch\share\
   move /Y torch\test\*.* libtorch\test\
   move /Y libtorch\bin\*.dll libtorch\lib\
   
   :: Create output under dist
   mkdir dist
   tar -cvaf dist/libtorch-win.zip -C libtorch *
   
   :: Cleanup raw data to save space
   rmdir /s /q libtorch
   ```

1. Now you can copy and extract `dist/libtorch-win.zip` file anywhere in your system

   - For C++ example, you can visit [Installing C++ Distributions of PyTorch](https://pytorch.org/cppdocs/installing.html)

   - You can just add following line to your `CMakeLists.txt` file.

      ```cmake
      set(CMAKE_PREFIX_PATH "C:/YOUR_CORRESPONDING_PATH/libtorch-win")
      ```