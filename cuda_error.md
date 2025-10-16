### ERROR: ###
```bash
from torch._C import *  # noqa: F403
ImportError: libcusparseLt.so.0: cannot open shared object file: No such file or directory
```
### SOLUTION: ###
Install `cusparselt` from this [link](https://developer.nvidia.com/cusparselt-downloads) 

---
### How to find CUDA?

```bash
sudo find / -name "libcudart.so.*" 2>/dev/null
```
<br> </br>
---
# Enabling CUDA Acceleration in Poetry Environments

When using GPU-accelerated Python libraries (e.g., PyTorch, TensorFlow, ONNX Runtime, JAX, etc.) in a [Poetry](https://python-poetry.org/) environment, you may encounter runtime errors like:

```
RuntimeError: Library libcublas.so.12 is not found or cannot be loaded
```

or

```
OSError: Unable to load any of {libcudnn_ops.so.9.1.0, libcudnn_ops.so.9.1, libcudnn_ops.so.9, libcudnn_ops.so}
```

These errors occur because **NVIDIA’s CUDA libraries** (cuBLAS, cuDNN, etc.)—even when installed via `pip`—are **not automatically discoverable** by the system’s dynamic linker in isolated Poetry environments.

This guide explains the root cause and provides a general, reusable solution.

---

## 🔍 Why This Happens

Many deep learning and scientific computing libraries rely on NVIDIA’s low-level CUDA libraries:
- **cuBLAS**: Basic linear algebra (e.g., matrix multiplications)
- **cuDNN**: Optimized primitives for deep neural networks
- **cuRAND**, **cuSOLVER**, etc. (less common for inference)

NVIDIA now publishes official **self-contained** CUDA library packages on PyPI:
- `nvidia-cudnn-cu12`
- `nvidia-cublas-cu12`
- `nvidia-cuda-runtime-cu12`
- etc.

When you install these via `pip` in a Poetry environment, the `.so` files are placed inside the virtual environment, e.g.:

```
<poetry_env>/lib/python3.10/site-packages/nvidia/cudnn/lib/
<poetry_env>/lib/python3.10/site-packages/nvidia/cublas/lib/
```

However, the Linux dynamic linker **does not search these paths by default**, so applications fail to load the required shared libraries at runtime—even though they are installed.

---

## ✅ General Solution

### 1. Install the required NVIDIA CUDA packages

For **CUDA 12.x** (recommended for modern GPUs and drivers):

```bash
poetry run pip install nvidia-cudnn-cu12 nvidia-cublas-cu12
```

> 💡 `nvidia-cudnn-cu12` usually pulls in `nvidia-cublas-cu12` automatically, but it’s safe to list both.

For **CUDA 11.x**, use the `-cu11` variants (e.g., `nvidia-cudnn-cu11`), but ensure compatibility with your framework and driver.

### 2. Add library paths to `LD_LIBRARY_PATH`

This is the **critical step**. You must tell the system where to find the `.so` files:

```bash
# Get Poetry environment path
ENV_PATH=$(poetry env info --path)

# Detect Python version dynamically
PYTHON_VER=$(python -c "import sys; print(f'python{sys.version_info.major}.{sys.version_info.minor}')")

# Prepend NVIDIA library paths
export LD_LIBRARY_PATH="$ENV_PATH/lib/$PYTHON_VER/site-packages/nvidia/cudnn/lib:$ENV_PATH/lib/$PYTHON_VER/site-packages/nvidia/cublas/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
```

> ✅ This ensures both cuDNN and cuBLAS are available. Add more paths (e.g., `cuda_runtime/lib`) if your app requires them.

### 3. Run your application

```bash
poetry run python your_gpu_script.py
```

It should now leverage GPU acceleration without library-loading errors.

---

## 🧰 Automation (Recommended)

To avoid repeating the `export` command, create reusable setup scripts.

### `setup_cuda.sh`
```bash
#!/bin/bash
ENV_PATH=$(poetry env info --path)
PYTHON_VER=$(python -c "import sys; print(f'python{sys.version_info.major}.{sys.version_info.minor}')")
export LD_LIBRARY_PATH="$ENV_PATH/lib/$PYTHON_VER/site-packages/nvidia/cudnn/lib:$ENV_PATH/lib/$PYTHON_VER/site-packages/nvidia/cublas/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
```

Make it executable:
```bash
chmod +x setup_cuda.sh
```

Usage:
```bash
source ./setup_cuda.sh
poetry run python your_script.py
```

### Universal runner (`run.sh`)
```bash
#!/bin/bash
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
source "$SCRIPT_DIR/setup_cuda.sh"
exec poetry run "$@"
```

Then simply:
```bash
./run.sh python train.py
./run.sh jupyter notebook
```

---

## ⚠️ Important Notes

- **Driver Compatibility**: Ensure your NVIDIA driver supports the CUDA version you’re using. Check with:
  ```bash
  nvidia-smi  # Top-right shows max supported CUDA version
  ```
  CUDA 12.x requires driver version ≥ 525.

- **No System CUDA Required**: This method uses **PyPI-distributed CUDA libraries**, so you **do not need** to install the full CUDA Toolkit (`cuda-toolkit-12-4`, etc.) system-wide.

- **Avoid Conflicts**: If you have multiple CUDA versions installed (e.g., via `apt`, Conda, or system paths), ensure `LD_LIBRARY_PATH` prioritizes the Poetry-managed libraries to prevent version mismatches.

- **Other Libraries**: Some applications may also need:
  - `nvidia-cuda-runtime-cu12` → provides `libcudart.so.12`
  - `nvidia-cuda-nvcc-cu12` → only needed for compilation, not runtime

  Add their `lib/` paths to `LD_LIBRARY_PATH` if required.

---

## ✅ Verification Script

Test that CUDA libraries load correctly:

```python
# test_cuda_libraries.py
import ctypes

required_libs = ["libcudnn.so.9", "libcublas.so.12"]

for lib in required_libs:
    try:
        ctypes.CDLL(lib)
        print(f"✅ {lib} loaded successfully")
    except OSError as e:
        print(f"❌ Failed to load {lib}: {e}")
        exit(1)
```

Run with:
```bash
source ./setup_cuda.sh
poetry run python test_cuda_libraries.py
```

---

With this setup, **any CUDA-accelerated Python application** will work reliably in Poetry-managed environments—without system-wide dependencies or complex CUDA toolkit installations. 🚀
