# HexDevice Common API Documentation

## Version Information

- **API Version**: 1.0
- **Protocol Version**: (1, 0)
- **Compatibility**: Requires HexDevice Python SDK v1.0 or later

## Table of Contents

1. [HexDeviceApi](#hexdeviceapi)
2. [DeviceBase](#devicebase)
3. [OptionalDeviceBase](#optionaldevicebase)
4. [MotorBase](#motorbase)

## HexDeviceApi

## `__init__`
```python
def __init__(self, ws_url: str = None, control_hz: int = 500, enable_kcp: bool = True, local_port: int = None, send_down_callback=None):
```
Creates the main HexDevice API runtime, validates the WebSocket endpoint, and launches the internal asyncio worker thread that manages device discovery and message dispatch.

Parameters:
- `ws_url`: WebSocket URL of the HexDevice server. Raises an `InvalidWSURLException` if the value is not supported. Set to `None` when using stream mode.
- `control_hz`: Frequency (Hz) for internal scheduling when processing device tasks. Defaults to 500 Hz.
- `enable_kcp`: Enables the accelerated KCP transport channel when available. When `True`, the API will negotiate the UDP tunnel, otherwise all traffic stays on the base WebSocket connection. Automatically disabled in stream mode.
- `local_port`: Explicit local UDP port for the KCP client. Use `None` to let the OS pick a free port automatically.
- `send_down_callback`: Optional callback function that is called when a message is sent down to the device. When provided, enables stream mode which bypasses all WebSocket/KCP transport. The caller is responsible for feeding data by invoking `_process_api_up()` directly and receives outgoing commands through this callback.

Examples:
```python
from hex_device import HexDeviceApi
api = HexDeviceApi(ws_url=args.url, control_hz=250, enable_kcp=True, local_port=52323)
```

## device_list
```python
@property
def device_list(self):
```
Returns a read-only device list containing all HEX series device class instances currently captured in the WebSocket.  

Examples:
```python
for device in api.device_list:
    if isinstance(device, Chassis):
        pass
    elif isinstance(device, Arm):
        pass
```

## optional_device_list
```python
@property
def optional_device_list(self):
```
Returns a read-only view of the optional device instances that were dynamically attached to registered primary devices (for example Hands peripherals). The list blocks all mutating methods so the SDK can keep ownership of lifecycle management.

Examples:
```python
for optional in api.optional_device_list:
    print(optional.device_id, optional.device_type)
```

## find_device_by_robot_type
```python
def find_device_by_robot_type(self, robot_type) -> Optional[DeviceBase]:
```
Matches device instances in the current device list based on the provided robot type. Returns `None` if no matching device is found.  
Examples:
```python
# RtArmArcherD6Y = 16;
archer = api.find_device_by_robot_type(16)
if archer is not None:
    print(f"Found device: {archer.name}")
```

## find_optional_device_by_id
```python
def find_optional_device_by_id(self, device_id: int) -> Optional[OptionalDeviceBase]:
```
Retrieves an optional device (secondary device) by the `device_id` reported in `SecondaryDeviceStatus`. Returns `None` if the device has not been discovered or removed.  
Examples:
```python
device = api.find_optional_device_by_id(device_id=1)
if device is not None:
    print(f"Found device: {device.name}")
```

## find_optional_device_by_robot_type
```python
def find_optional_device_by_robot_type(self, robot_type) -> Optional[List[OptionalDeviceBase]]:
```
Finds optional devices by robot type. Returns a list of matching optional devices or `None` if no devices are found.  
Examples:
```python
devices = api.find_optional_device_by_robot_type(robot_type=1)
if devices is not None:
    for device in devices:
        print(f"Found device: {device.name}, ID: {device.device_id}")
```

## get_device_task_status
```python
def get_device_task_status(self) -> Dict[str, Any]:
```
Gets a high-level snapshot of the internal task scheduler. The returned dictionary contains:
- `total_devices`: Number of devices currently tracked.
- `active_tasks`: Total count of periodic tasks still running.
- `device_tasks`: Mapping keyed by device name with `device_id`, `device_type`, `robot_type`, and task state flags (`task_done`, `task_cancelled`).  
Examples:
```python
status = api.get_device_task_status()
print(f"Total devices: {status['total_devices']}")
print(f"Active tasks: {status['active_tasks']}")
for device_name, task_info in status['device_tasks'].items():
    print(f"Device {device_name}: task_done={task_info['task_done']}, task_cancelled={task_info['task_cancelled']}")
```

## close
```python
def close(self):
```
Cleans up all asynchronous threads and closes the API interface.  
Examples:
```python
try:
    while not api.is_api_exit():
        pass
except KeyboardInterrupt:
    print("Received Ctrl-C.")
finally:
    api.close()
```

## is_api_exit
```python
def is_api_exit(self) -> bool:
```
Checks if the API has exited.  
Examples:
```python
try:
    while not api.is_api_exit():
        pass
except KeyboardInterrupt:
    print("Received Ctrl-C.")
finally:
    api.close()
```

## get_raw_data
```python
def get_raw_data(self) -> Tuple[Optional[public_api_up_pb2.APIUp], int]:
```
Returns a tuple of the oldest buffered `APIUp` protobuf message and the remaining queue length. The first element is `None` if the buffer is empty. Internally the API maintains a sliding window buffer (maximum length `RAW_DATA_LEN`, equal to 50 frames). By consuming the queue frequently you can reconstruct a lossless real-time stream.

**Parameters:**
None

**Returns:**
- `Tuple[Optional[public_api_up_pb2.APIUp], int]`: A tuple containing:
  - The oldest buffered `APIUp` protobuf message, or `None` if the buffer is empty
  - The remaining queue length in the buffer

**Notes:**
- The buffer has a maximum capacity of 50 frames. When the buffer is full, the oldest message is automatically removed to make room for new messages.
- This method is thread-safe and can be called from multiple threads simultaneously.
- The `APIUp` message contains all device status information, including base status, arm status, and secondary device status.

Examples:
```python
# Basic usage example
while not api.is_api_exit():
    (data, num) = api.get_raw_data()
    if data is not None:
        # Process raw data
        if data.HasField('base_status'):
            # Handle chassis status
            pass
        elif data.HasField('arm_status'):
            # Handle arm status
            pass

# Stream mode usage example
# 1. Define callback function to handle downlink messages
def send_down_callback(message):
    """Handle messages sent to the device"""
    # Process the message here, e.g., send over custom protocol
    print(f"Sending message of length: {len(message)}")

# 2. Create API instance with stream mode enabled
api = HexDeviceApi(ws_url=None, send_down_callback=send_down_callback)

# 3. Simulate data input (in real application, get from external data source)
def simulate_data_input(api):
    """Simulate data input to API"""
    while not api.is_api_exit():
        # Here you should get APIUp messages from actual data source
        # e.g., from network, file, or other devices
        # Then call api._process_api_up(api_up_message)
        time.sleep(0.01)  # Simulate 100Hz data input

# 4. Start data input thread
import threading
import time
data_thread = threading.Thread(target=simulate_data_input, args=(api,))
data_thread.daemon = True
data_thread.start()

# 5. Process output data
while not api.is_api_exit():
    (data, num) = api.get_raw_data()
    if data is not None:
        # Process received data
        pass
```

# MotorBase

## Overview
The `MotorBase` class provides common motor control functionality for devices with motors. It is inherited by device classes that require motor control capabilities.

## Key Features
- Motor status monitoring
- Common motor control operations
- Temperature monitoring

## Documentation
For complete MotorBase documentation including all methods and usage examples, please refer to [API-MotorBase](API-Motorbase).

# OptionalDeviceBase

## `__init__`
```python
def __init__(self, read_only: bool, name: str, device_id, device_type, send_message_callback=None):
```
Initializes an optional device base class. These devices are matched by `device_id` (from `SecondaryDeviceStatus`) rather than `robot_type` and are used for processing optional fields in APIUp messages.

**Parameters:**
- `read_only` (bool): Whether the device is read-only
- `name` (str): Device name
- `device_id`: Device ID from SecondaryDeviceStatus
- `device_type`: Device type from SecondaryDeviceStatus
- `send_message_callback` (callable, optional): Callback function for sending messages

**Examples:**
```python
# Usually called internally by HexDeviceApi when creating optional devices
# Users typically don't need to call this directly
```

## get_device_summary
```python
def get_device_summary(self) -> Dict[str, Any]:
```
Gets the device status summary including name and assigned device ID.

**Returns:**
- `Dict[str, Any]`: Dictionary containing device status information with keys:
  - `name`: Device name
  - `device_id`: Device ID from SecondaryDeviceStatus

**Examples:**
```python
summary = device.get_device_summary()
print(f"Device: {summary['name']}")
print(f"Device ID: {summary['device_id']}")
```

# DeviceBase

## `__init__`
```python
def __init__(self, name: str = "", send_message_callback=None):
```
Automatically called by HexDeviceApi, passing in the device name and callback function for sending WebSocket messages.

**Parameters:**
- `name` (str, optional): Device name, defaults to "Device"
- `send_message_callback` (callable, optional): Callback function for sending messages

## start
```python
def start(self):
```
Starts device control. Sets the internal control flag to enable sending control commands.  
Examples:
```python
device.start()
```

## stop
```python
def stop(self):
```
Stops device control. Sets the internal control flag to disable sending control commands.  
Examples:
```python
device.stop()
```

## get_device_summary
```python
def get_device_summary(self) -> Dict[str, Any]:
```
Gets the current device status summary.

**Returns:**
- `Dict[str, Any]`: Dictionary containing device status information with keys:
  - `name`: Device name

**Examples:**
```python
summary = device.get_device_summary()
print(f"Device name: {summary['name']}")
```
