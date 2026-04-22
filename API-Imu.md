# Imu API Documentation

<!-- 
## Version Information

- **API Version**: 1.0
- **Protocol Version**: (1, 0)
- **Compatibility**: Requires HexDevice Python SDK v1.0 or later
 -->

## Table of Contents

1. [Overview](#overview)
2. [Class Definition](#class-definition)
3. [Initialization](#__init__)
4. [Data Methods](#data-methods)
   - [get_imu_data](#get_imu_data)
5. [Inherited Methods](#inherited-methods)


## Overview

The `Imu` class inherits from [OptionalDeviceBase](API-Common#OptionalDeviceBase), primarily implementing IMU (Inertial Measurement Unit) data reading and status management. This class processes the optional `imu_data` field from APIUp messages.

Supported device types:
- `SdtImuY200`: IMU Y200 device type

## Class Definition
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

```proto
message ImuAcceleration {
    float ax = 1; // m/s^2
    float ay = 2; // m/s^2
    float az = 3; // m/s^2
}

message ImuAngularVelocity {
    float wx = 1; // rad/s
    float wy = 2; // rad/s
    float wz = 3; // rad/s
}

message ImuQuaternion {
    float qx = 1; // unitless
    float qy = 2; // unitless
    float qz = 3; // unitless
    float qw = 4; // unitless
}

message ImuData {
    ImuAcceleration acceleration = 1;
    ImuAngularVelocity angular_velocity = 2;
    ImuQuaternion quaternion = 3;
}
```

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
<!-- 

## Best Practices

### General Recommendations

1. **Check for None values**
   - IMU data may contain None values if sensor is not initialized
   - Always check for None before processing data

2. **Use appropriate data retrieval frequency**
   - IMU data is typically available at 250Hz
   - Don't poll more frequently than necessary

3. **Handle quaternion data properly**
   - Quaternion is in [w, x, y, z] format
   - Ensure proper normalization when converting to other representations

### Data Processing Tips

1. **Use filtering for noisy data**
   - IMU data may contain noise
   - Consider applying low-pass filters for smoothing

2. **Convert units as needed**
   - Acceleration is in m/s²
   - Angular velocity is in rad/s

## Troubleshooting

### Common Issues and Solutions

1. **IMU Data is None**
   - **Symptom**: IMU data fields return None
   - **Cause**: Sensor not initialized or communication issue
   - **Solution**: Check device connection and initialization

2. **No IMU Device Found**
   - **Symptom**: `find_optional_device_by_id` or `find_optional_device_by_robot_type` returns None
   - **Cause**: IMU device not connected or not registered
   - **Solution**: Verify IMU device is properly connected and configured

### Debugging Tips

1. **Enable verbose logging**
   - Set logging level to DEBUG to see detailed communication

2. **Monitor device status**
   - Regularly check `get_imu_summary()` for device health

3. **Test with simple data retrieval**
   - Start with basic `get_imu_data()` calls to verify functionality
 -->