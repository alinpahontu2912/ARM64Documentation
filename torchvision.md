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





