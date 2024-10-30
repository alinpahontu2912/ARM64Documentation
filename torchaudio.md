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
