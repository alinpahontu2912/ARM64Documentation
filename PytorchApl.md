# Building PyTorch with APL

## Prerequisites
1. **Install necessary tools:**
   - Git
   - Visual Studio 2022 or VS Build Tools 2022
   - Python 3.12 Arm64 from [Python Releases for Windows](https://www.python.org/downloads/windows/)
   - Git for Windows from [git-scm.com](https://git-scm.com/downloads)
   
   - If using Build Tools, ensure "ARM64/ARM64EC build tools" are installed from individual components (known stable version: ARM64/ARM64EC v14.36 - 17.6).


2. **Register ninja.exe in your PATH environment variable:**
   - Download from [Ninja Releases](https://github.com/ninja-build/ninja/releases).

3. **Download and install APL from [Arm Performance Libraries](https://developer.arm.com/tools-and-software/server-and-hpc/compile/arm-performance-libraries/downloads):**
   - **Newest version (APL v24.04):**
     - Download and run the installer package. The `%ARMPL_DIR%` environment variable will be added to your PATH by default.


## Setup
4. **Open Arm64 Native Tools Command Prompt for VS2022:**
   - `"C:\Program Files\Microsoft Visual Studio\2022\Preview\VC\Auxiliary\Build\vcvarsarm64.bat"`
   - `"C:\Program Files\Microsoft Visual Studio\2022\Enterprise\VC\Auxiliary\Build\vcvarsall.bat" arm64`
   - `"C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvarsall.bat" arm64`
   - `"C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvarsall.bat" arm64`

   - To activate a specific MSVC version:
     - `"C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvarsall.bat" arm64 -vcvars_ver=14.36` (stable version)
     - `"C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvarsall.bat" arm64 -vcvars_ver=14.40` (latest version)

## Environment Variables and Dependencies
5. Set the following environment variables:
   - `set BLAS=APL`
   - `set REL_WITH_DEB_INFO=1`
   - `set CMAKE_BUILD_TYPE=RelWithDebInfo`
   - `set USE_LAPACK=1`

6. **(Optional) Use sccache to speed up build times:** 
   - Download the latest sccache from [sccache releases](https://github.com/mozilla/sccache/releases).
   - Add `SCCACHE_CACHE_SIZE=30G` to system environments.
   - Add the sccache binary folder to the PATH.
   - Set the following variables:
     - `set CMAKE_C_COMPILER_LAUNCHER=sccache`
     - `set CMAKE_CXX_COMPILER_LAUNCHER=sccache`
 
## Cloning and Setting Up PyTorch
7. Clone the PyTorch repository and setup submodules:
   - `git clone --recursive -b apl-tests https://github.com/iremyux/pytorch.git`
   - `cd pytorch`
   - `git submodule sync`
   - `git submodule update --init --recursive --jobs 4`
   - You need to modify the `aten\src\ATen\native\MathBitsFallback.h` file by disabling the optimization of the fallback_impl :
```c
#pragma optimize("", off) // ADD THIS BEFORE the method, ~line 27 
void fallback_impl(const c10::OperatorHandle& op, DispatchKeySet dispatch_keys, torch::jit::Stack* stack) {
...
}
#pragma optimize("", on) // ADD THIS AFTER the method, ~line 151
 ```
8. **Create a virtual environment and install requirements:**
   - `python -m venv venv`
   - `echo * > ./venv/.gitignore`
   - `call .\venv\Scripts\activate`
   - Install _numpy_ and _Scipy_ wheels from [win_arm64-wheels](https://github.com/cgohlke/win_arm64-wheels) in the environment before starting the build
   - `pip install -r requirements.txt`

9. **Install PyTorch:**
   - `python setup.py install`

## Running the Tests
- **Tests are located in:** `pytorch/test`
- **Using VSCode:**
    - The `launch.json` file in the `.vscode` directory simplifies running tests without the command line.
    - We are using the _Python C++ debugger by BeniBenj_ extension for debugging both python and C++ side.


### Next steps:
- After the build is finished run this command:
   - *pip install pytest==8.1.1 pytest-xdist==3.5.0 pytest-shard pytest-rerunfailures==13.0 pytest-flakefinder pytest-pytorch expecttest hypothesis xdoctest* 
- Under `launch.json` file, make sure the Test Single configuration looks like this: `test_fn_fwgrad_bwgrad_atanh_cpu_complex128` is the test we used to provide the screenshots 

```json {
    "name": "Test Single",
    "type": "python",
    "request": "launch",
    "program": "test_ops_fwd_gradients.py",
    "cwd": "${workspaceFolder}/test",
    "args": [
        "-vvvv",
        "-rfEsxXP",
        "-p", "no:xdist",
        "--use-pytest",
        "-k", "test_fn_fwgrad_bwgrad_atanh_cpu_complex128"
    ],
    "console": "integratedTerminal",
    "justMyCode": false
} 
```
###
- Add debug points to these lines in the aten\src\ATen\native\MathBitsFallback.h file(approximately line 132, 134):
 
    - `op.redispatchBoxed(dispatch_keys & c10::DispatchKeySet(DispatchKeySet::FULL_AFTER, key), stack);`
 
    - `TORCH_INTERNAL_ASSERT(mutable_inputs_with_their_clones.size() <= 1);`
- run `Test C++` in debug mode
- these are the values to watch:
   - (c10::complex<double>*)stack[0][0].payload.as_tensor.impl_.target_->storage_.storage_impl_.target_->data_ptr_.ptr_.data_
   - (c10::complex<double>*)stack[0][0].payload.as_tensor.impl_.target_->storage_.storage_impl_.target_->data_ptr_.ptr_.data_+1
   - (c10::complex<double>*)stack[0][0].payload.as_tensor.impl_.target_->storage_.storage_impl_.target_->data_ptr_.ptr_.data_+2
   - (c10::complex<double>*)stack[0][0].payload.as_tensor.impl_.target_->storage_.storage_impl_.target_->data_ptr_.ptr_.data_+3
   - (c10::complex<double>*)stack[0][0].payload.as_tensor.impl_.target_->storage_.storage_impl_.target_->data_ptr_.ptr_.data_+4

