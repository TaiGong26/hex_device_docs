# hex_device Installation Guide
### Prerequisites
- **Python 3.9 or higher**
- Anaconda Distribution (recommended for beginners) - includes Python, NumPy, and commonly used scientific computing packages

---

## Installation Options
Choose one of the following options based on your usage needs:

### Option 1: Direct Pip Installation (For Regular Users)
Install the library directly from PyPI for regular usage (no source code required):
```bash
python3 -m pip install hex_device
```

### Option 2: Source Code Installation (Local Build)
Install the library from local source code (for customized builds):

**Prerequisites:**
- `protoc` v27.1 required. See [**Install Protobuf Compiler**](#install-protobuf-compiler-protoc)

```bash
python3 -m pip install .
```

### Option 3: Direct Add
Use the library directly from source (for development/debugging, no installation required):

**Prerequisites:**
- `protoc` v27.1 required. See [**Install Protobuf Compiler**](#install-protobuf-compiler-protoc)

1. **Install project dependencies**
    ```bash
    python3 -m pip install -r requirements.txt
    ```

2. **Add library path to your Python script**
    ```python
    import sys
    # Replace <your project path> with the actual path to hex_device_python
    sys.path.insert(1, '<your project path>/hex_device_python')
    sys.path.insert(1, '<your project path>/hex_device_python/hex_device/generated')
    ```

---

### Install Protobuf Compiler (`protoc`)
**Required for compiling protocol buffer messages**

**Version: ==27.1 required**

```bash
# For Linux arm64
wget https://github.com/protocolbuffers/protobuf/releases/download/v27.1/protoc-27.1-linux-aarch_64.zip

sudo unzip protoc-27.1-linux-aarch_64.zip -d /usr/local
rm protoc-27.1-linux-aarch_64.zip

# Verify installation
protoc --version  # Should show : libprotoc 27.1
```
---

### Compile Protocol Buffer Messages
**Execute this step before installation/usage**
```bash
# Create generated directory and compile proto files
mkdir -p ./hex_device/generated
protoc --proto_path=proto-public-api --python_out=hex_device/generated proto-public-api/*.proto && cp ./proto-public-api/version.py ./hex_device/generated/version.py
```

---
