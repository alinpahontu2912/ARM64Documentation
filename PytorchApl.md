# Building torch suite Windows on ARM64

## Pytorch
1. **Install necessary tools:**
   - Visual Studio 2022 or VS Build Tools 2022 with the **Desktop C++ Development kit**
   - Python 3.12 Arm64 from [Python Releases for Windows](https://www.python.org/downloads/windows/)
   - Git for Windows from [git-scm.com](https://git-scm.com/downloads)
   - If using **Build Tools**, make sure "ARM64/ARM64EC build tools" are installed from individual components


2. **Register ninja.exe in your PATH environment variable:**
   - Download from [Ninja Releases](https://github.com/ninja-build/ninja/releases) and add it to PATH 

3. **Download and install APL from [Arm Performance Libraries](https://developer.arm.com/tools-and-software/server-and-hpc/compile/arm-performance-libraries/downloads):**
   - **Newest version (APL v24.10):**
     - Download and run the the msi installer. The `%ARMPL_DIR%` environment variable will be added to your PATH by default.
     - Note: Oldest supported APL version is 24.04

### Cloning and Setting Up PyTorch
4. Clone the PyTorch repository and setup submodules:
```
   git clone --recursive https://github.com/pytorch/pytorch
   cd pytorch
   # if you are updating an existing checkout
   git submodule sync
   git submodule update --init --recursive
```
5. **Create a virtual environment and install requirements:**
```
   # Create virtual python environment (Optional)
   python -m venv venv
   echo * > ./venv/.gitignore
   call .\venv\Scripts\activate
   # Install requirements
   pip install -r requirements.txt
```
   - Note: A Python virtual environment helps isolate dependencies to prevent conflicts with other projects. It’s especially helpful if you're working on multiple projects. However, if you're fine with installing everything globally and don’t need separation, you can skip the venv.

6. **Set up Arm64 Native Tools Command Prompt for VS2022:**
   - Based on what you installed earlier, type:
   - `"C:\Program Files\Microsoft Visual Studio\2022\Preview\VC\Auxiliary\Build\vcvarsarm64.bat"`
   - `"C:\Program Files\Microsoft Visual Studio\2022\Enterprise\VC\Auxiliary\Build\vcvarsall.bat" arm64`
   - `"C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvarsall.bat" arm64`
   - `"C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvarsall.bat" arm64`

### Environment Variables and Dependencies
7. Set the following environment variables:
```
   set BLAS=APL
   set REL_WITH_DEB_INFO=1
   set CMAKE_BUILD_TYPE=RelWithDebInfo
   set USE_LAPACK=1
```
8. **(Optional) Use sccache to speed up build times:** 
   - Download the latest sccache from [sccache releases](https://github.com/mozilla/sccache/releases).
   - Add `SCCACHE_CACHE_SIZE=30G` to system environments.
   - Add the sccache binary folder to the PATH.
   - Set the following variables in the build environment:
     - `set CMAKE_C_COMPILER_LAUNCHER=sccache`
     - `set CMAKE_CXX_COMPILER_LAUNCHER=sccache`
   - Note: Using sccache can significantly speed up PyTorch builds by caching previous compilation results. This is especially useful if you're making frequent code changes, as it avoids re-compiling unchanged files.
 

6. **Build PyTorch:**
```
   python setup.py install
```

## Building Libtorch 

## Building Torchaudio
### Prerequisites
1. **Install necessary tools:**
   - Visual Studio 2022 or VS Build Tools 2022 with the **Desktop C++ Development kit**
   - Python 3.12 Arm64 from [Python Releases for Windows](https://www.python.org/downloads/windows/)
   - Git for Windows from [git-scm.com](https://git-scm.com/downloads)
   - If using **Build Tools**, make sure "ARM64/ARM64EC build tools" are installed from individual components
   - Download ninja from [Ninja Releases](https://github.com/ninja-build/ninja/releases) and add it to PATH
   - Make sure you have a pytorch wheel file, or pytorch is already installed in your environment/system

### Cloning and Setting Up Torchaudio
2. Clone the Torchvision repository:
```
   git clone https://github.com/pytorch/audio 
   cd audio
```
3. **Create a virtual environment and install dependencies**
```
   # Create virtual python environment (Optional)
   python -m venv venv
   echo * > ./venv/.gitignore
   call .\venv\Scripts\activate 
   # install requirements
   pip install -r requirements.txt
```
   - Note: Currently there are some issues with the soundfile wheel, so we recommend installing it from [here](https://github.com/cgohlke/win_arm64-wheels/releases/tag/v2023.4.1) and not from pip. If your python version is different, please rename the wheel file accordingly
      - soundfile-0.12.1-cp312-cp312-win_arm64.whl for python 3.12
      - soundfile-0.12.1-cp311-cp311-win_arm64.whl for python 3.11
   
4. **Set up Arm64 Native Tools Command Prompt for VS2022:**
   - Based on what you installed earlier, type:
   - `"C:\Program Files\Microsoft Visual Studio\2022\Preview\VC\Auxiliary\Build\vcvarsarm64.bat"`
   - `"C:\Program Files\Microsoft Visual Studio\2022\Enterprise\VC\Auxiliary\Build\vcvarsall.bat" arm64`
   - `"C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvarsall.bat" arm64`
   - `"C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvarsall.bat" arm64`

### Build Torchvision
5. **Set the following environment variables:**
```
   set USE_FFMPEG=0
```

6. **Run:** 
```
   python setup.py install
```

### Notes
   - Currently there is no official FFMPEG suport. We understand it is an important feature and are working towards bringing it for Windows on ARM64.

## Building Torchvision
### Prerequisites
1. **Install necessary tools:**
   - Visual Studio 2022 or VS Build Tools 2022 with the **Desktop C++ Development kit**
   - Python 3.12 Arm64 from [Python Releases for Windows](https://www.python.org/downloads/windows/)
   - Git for Windows from [git-scm.com](https://git-scm.com/downloads)
   - If using **Build Tools**, make sure "ARM64/ARM64EC build tools" are installed from individual components
   - Download ninja from [Ninja Releases](https://github.com/ninja-build/ninja/releases) and add it to PATH
   - Install vcpkg, instructions [here](https://learn.microsoft.com/en-us/vcpkg/get_started/get-started?pivots=shell-powershell#1---set-up-vcpkg)
   - Make sure you a pytorch wheel file, or pytorch is already installed in your environment/system

### Downloading external libraries
2. After installing vcpkg,you need to install the following dependencies:
   - vcpkg install libjpeg-turbo 
   - vcpkg install libwebp 

### Clone and set up PyTorch
3. Clone the Torchvision repository:
```
   git clone --recursive https://github.com/pytorch/vision.git
   cd vision
```
4. Create a virtual environment and install dependencies
```
   # Create virtual python environment (Optional)
   python -m venv venv
   echo * > ./venv/.gitignore
   call .\venv\Scripts\activate   
   # install requirements
   pip install numpy pillow 
```
   - Note: You can use the same virtual environment used to build pytorch or you can create a new one. In this case, you will need to install pytorch using a wheel file you generated.

5. **Set up Arm64 Native Tools Command Prompt for VS2022:**
   - Based on what you installed earlier, type:
   - `"C:\Program Files\Microsoft Visual Studio\2022\Preview\VC\Auxiliary\Build\vcvarsarm64.bat"`
   - `"C:\Program Files\Microsoft Visual Studio\2022\Enterprise\VC\Auxiliary\Build\vcvarsall.bat" arm64`
   - `"C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvarsall.bat" arm64`
   - `"C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvarsall.bat" arm64`

6. **Set the following environment variables:**
```
   set DISTUTILS_USE_SDK=1
   set TORCHVISION_LIBRARY=<path_to_vcpkg>/packages/libwebp/lib;<path_to_vcpkg>/packages/libjpeg-turbo/lib;
   set TORCHVISION_LIBRARY=<path_to_vcpkg>/packages/libjpeg-turbo/include; <path_to_vcpkg>/packages/libwebp/include;
```
   Note: Replace <path_to_vcpkg> with the absolute path to your vcpkg installation folder

### Build Torchvision
7. **Build the package**
```
   python setup.py install
```

### Notes
   - Currently there is no PNG file suport. We understand it is an important feature and are working towards bringing it for Windows on ARM64.





