# Building torch suite Windows on ARM64

> Note: Commit test

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
