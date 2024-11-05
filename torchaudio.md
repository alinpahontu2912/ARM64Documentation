## Building Torchaudio
### Prerequisites
1. **Install necessary tools:**
   - Visual Studio 2022 or VS Build Tools 2022 with the **Desktop C++ Development kit**
   - Python 3.12 Arm64 from [Python Releases for Windows](https://www.python.org/downloads/windows/)
   - Git for Windows from [git-scm.com](https://git-scm.com/downloads)
   - If using **Build Tools**, make sure "ARM64/ARM64EC build tools" are installed from individual components
   - Download and set-up vcpkg, instructions [here](https://learn.microsoft.com/en-us/vcpkg/get_started/get-started?pivots=shell-powershell)
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

# Building for Windows on ARM64

1. [Prerequisites](#prerequisites)
1. [Cloning Torchaudio](#cloning-pytorch)
1. [Preparing Environment](#preparing-environment)
1. [Setting Environment Variables](#setting-environment-variables)
1. [Build](#build)

## Prerequisites

1. [Build Tools for Visual Studio 2022](https://aka.ms/vs/17/release/vs_BuildTools.exe) **OR** any version of [Visual Studio 2022](https://visualstudio.microsoft.com/downloads/)
   
   - Under **Workloads**, select **Desktop development with C++**

   - Under **Individual Components**, select **ARM64/ARM64EC build tools (latest)**

1. [Python 3.12 Windows installer (ARM64)](https://www.python.org/downloads/windows/)

1. Have Pytorch already installed in your environment **OR** a readily-available Pytorch wheel file (.whl)

## Cloning TorchAudio

1. Clone the Torchaudio repository (or your fork)

   ```cmd
   git clone  https://github.com/pytorch/audio.git
   cd audio
   ```

## Downloading Ffmpeg dependency

1. Open a command prompt and set up vcpkg

   Run these commands:
   ```cmd
   git clone https://github.com/microsoft/vcpkg.git
   git checkout f00e89ae1903b95301065da48c84fbaf1cb680ce ## This includes the latest compatible FFmpeg version 6
   cd vcpkg
   bootstrap-vcpkg.bat
   ```

1. Download ffmpeg libraries and add it to PATH

   ```cmd
   vcpkg install ffmpeg[ffmpeg]:arm64-windows
   ```

   - After the library is ready, check the package installation folder. It will look like this: **<vcpkg_install_path>\packages\ffmpeg_arm64-windows\tools\ffmpeg**.
   - Make sure that you can see and run the ffmpeg executable from this folder
   - Add the path to the executable to System Variables, under Path
   
   **(TEST Ffmpeg installation)** 
   - Open a new command prompt, type ffmpeg and check if it will run


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

   When building in an environment without pytorch already installed, a wheel file is needed:  
   
   ```cmd
   pip install <path_to_torch_wheel>/torch-FOO+gitBAR-cp312-cp312-win_arm64.whl ## Only when building in an environment without pytorch already installed
   
   pip install -r requirements.txt ## assuming you have the torch package installed in your environments already
   ```

   **Note**: Currently there are some issues with the soundfile wheel, so we recommend installing it from [here](https://github.com/cgohlke/win_arm64-wheels/releases/tag/v2023.4.1) and not from pip.
   
   If your python version is different, please rename the wheel file accordingly
      - **soundfile-0.12.1-cp312-cp312-win_arm64.whl** for python 3.12
      - **soundfile-0.12.1-cp311-cp311-win_arm64.whl** for python 3.11
   
   You can then install it to your environment:
   ```cmd
   pip install soundfile-0.12.1-cp312-cp312-win_arm64.whl ## for python 3.12
   ```

   **OR**

   ```cmd
   pip install soundfile-0.12.1-cp311-cp311-win_arm64.whl ## for python 3.11
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
1. Set the following environment variable:
   ```cmd
   set FFMPEG_ROOT = <path_to_vcpkg>/packages/ffmpeg_arm64-windows 
   ```

## Build

1. **OPTION 1 - Generating and Installing Wheel File** (for distribution or installation on other systems):

   Call following command to start build and generate output
   ```
   python setup.py bdist_wheel
   ```
   Output file will be under `dist` folder with `.whl` extension.
   
   Format will look like `torch-FOO+gitBAR-cp312-cp312-win_arm64.whl`

   Now you can copy and install this `.whl` file anywhere in your system by calling

   ```
   pip install torch-YOURVERSION-cp312-cp312-win_arm64.whl
   ```

1. **OPTION 2 - Directly Installing to Local Environment** (for development on your system only)

   Run one of the following commands to initiate the build:

   - Install the package in development mode, ideal when making frequent changes to codebase
   ```
   python setup.py develop 
   ```

   - Install the package for regular use, rather than ongoing development
   ```
   python setup.py install 
   ```