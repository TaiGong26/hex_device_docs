### Prerequisites

- **Python 3.9 or higher**
- Anaconda Distribution (recommended for beginners) - includes Python, NumPy, and commonly used scientific computing packages

### Option 1: Package Installation
To install the library in your Python environment through pip:
```
python3 -m pip install hex_device
```

Or install from source code:

```
python3 -m pip install .
```

**Note:** This library requires newer protoc. If compilation fails, please try to install protoc-27.1 using the binary installation method below.

   **Installing protoc-27.1 through binary:**
   ```bash
   # For Linux x86_64
   wget https://github.com/protocolbuffers/protobuf/releases/download/v27.1/protoc-27.1-linux-x86_64.zip
   sudo unzip protoc-27.1-linux-x86_64.zip -d /usr/local
   rm protoc-27.1-linux-x86_64.zip
   
   # For Linux arm64
   wget https://github.com/protocolbuffers/protobuf/releases/download/v27.1/protoc-27.1-linux-aarch_64.zip
   sudo unzip protoc-27.1-linux-aarch_64.zip -d /usr/local
   rm protoc-27.1-linux-aarch_64.zip
   
   #  Verify installation
   protoc --version  # Should show libprotoc 27.1
   ```

### Option 2: Direct Add

If you prefer to run the library without installing it in your Python environment:

1. **Compile Protocol Buffer messages:**
   ```bash
   mkdir ./hex_device/generated
   protoc --proto_path=proto-public-api --python_out=hex_device/generated proto-public-api/*.proto
   ```

2. **Install dependencies:**
```bash
python3 -m pip install -r requirements.txt
```

3. **Add the library path to your script:**
Add the library path to your script:
```python
import sys
sys.path.insert(1, '<your project path>/hex_device_python')
sys.path.insert(1, '<your project path>/hex_device_python/hex_device/generated')
```