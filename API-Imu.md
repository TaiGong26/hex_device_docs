The `Imu` class inherits from [OptionalDeviceBase](API-Common#OptionalDeviceBase), primarily implementing IMU (Inertial Measurement Unit) data reading and status management. This class processes the optional `imu_data` field from APIUp messages.

Supported device types:
- `SdtImuY200`: IMU Y200 device type

# Imu
```python
class Imu(OptionalDeviceBase):
```

The common function can be found in: [OptionalDeviceBase](API-Common#OptionalDeviceBase).

## `__init__`
```python
def __init__(self, device_id, device_type, send_message_callback, name: str = "Imu", control_hz: int = 250, read_only: bool = True):
```
Automatically called by HexDeviceApi to initialize the Imu device.

**Parameters:**
- `device_id`: Device ID from SecondaryDeviceStatus
- `device_type`: Device type (SecondaryDeviceType enum, e.g., SdtImuY200)
- `send_message_callback`: Callback function for sending messages
- `name` (str, optional): Device name, defaults to "Imu"
- `control_hz` (int, optional): Control frequency in Hz, defaults to 250
- `read_only` (bool, optional): Whether this device is read-only, defaults to True

Examples:
```python
# Usually called internally by HexDeviceApi when creating IMU devices
# Users typically don't need to call this directly
```

## get_imu_data
```python
def get_imu_data(self) -> Dict[str, Any]:
```
Gets the current IMU data including acceleration, angular velocity, and quaternion.

**Returns:**
- `Dict[str, Any]`: Dictionary containing IMU data with keys:
  - `acceleration`: Acceleration data (m/s²)
  - `angular_velocity`: Angular velocity data (rad/s)
  - `quaternion`: Quaternion data [w, x, y, z]

Examples:
```python
imu_data = imu.get_imu_data()
if imu_data['acceleration'] is not None:
    print(f"Acceleration: {imu_data['acceleration']}")
    print(f"Angular velocity: {imu_data['angular_velocity']}")
    print(f"Quaternion: {imu_data['quaternion']}")
```

## get_imu_summary
```python
def get_imu_summary(self) -> Dict[str, Any]:
```
Gets comprehensive IMU device summary including device type and IMU data.

**Returns:**
- `Dict[str, Any]`: Dictionary containing:
  - `imu_type`: Device type (SecondaryDeviceType enum value)
  - `imu_data`: Dictionary containing acceleration, angular_velocity, and quaternion

Examples:
```python
summary = imu.get_imu_summary()
print(f"IMU type: {summary['imu_type']}")
print(f"IMU data: {summary['imu_data']}")
```

## Inherited Methods

The `Imu` class inherits all methods from `OptionalDeviceBase`, including:

### From OptionalDeviceBase:
- `get_device_summary()` - Get device status summary (name and device_id)

**Examples:**
```python
# Get device summary
summary = imu.get_device_summary()
print(f"Device name: {summary['name']}")
print(f"Device ID: {summary['device_id']}")
```

## Usage Example

```python
from hex_device import HexDeviceApi
from hex_device.generated import public_api_types_pb2

# Create API instance
api = HexDeviceApi(ws_url="ws://localhost:8080")

# Find IMU device by device ID
imu = api.find_optional_device_by_id(device_id=1)

# Or find by device type
imu_devices = api.find_optional_device_by_robot_type(robot_type=public_api_types_pb2.SecondaryDeviceType.SdtImuY200)

if imu is not None:
    # Get IMU data
    imu_data = imu.get_imu_data()
    if imu_data['acceleration'] is not None:
        print(f"Acceleration: {imu_data['acceleration']}")
        print(f"Angular velocity: {imu_data['angular_velocity']}")
        print(f"Quaternion: {imu_data['quaternion']}")
    
    # Get IMU summary
    summary = imu.get_imu_summary()
    print(f"IMU type: {summary['imu_type']}")
    print(f"IMU data: {summary['imu_data']}")
    
    # Get device summary
    device_summary = imu.get_device_summary()
    print(f"Device name: {device_summary['name']}")
    print(f"Device ID: {device_summary['device_id']}")
```
