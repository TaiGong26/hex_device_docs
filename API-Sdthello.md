# SdtHello API Documentation

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
   - [has_new_data](#has_new_data)
   - [get_simple_motor_status](#get_simple_motor_status)
   - [get_joint_limits](#get_joint_limits)
   - [get_hello_summary](#get_hello_summary)
5. [Control Methods](#control-methods)
   - [set_rgb_stripe_command](#set_rgb_stripe_command)
6. [Inherited Methods](#inherited-methods)
7. [Usage Example](#usage-example)


## Overview

The `SdtHello` class inherits from [`OptionalDeviceBase`](API-Common.md#OptionalDeviceBase) and [`MotorBase`](API-Motorbase.md), primarily implementing Hello device data reading and RGB stripe control. This class processes the optional `hello1j1t4b_status` field from APIUp messages.

Supported device types:
- `SdtHello1J1T4BV1`: Hello1J1T4B V1 device type

## Class Definition
```python
class SdtHello(OptionalDeviceBase, MotorBase):
```

The common function can be found in: [OptionalDeviceBase](API-Common.md#OptionalDeviceBase) and [MotorBase](API-Motorbase.md).

## `__init__`
```python
def __init__(self, device_id, device_type, proto_version: tuple[int, int], send_message_callback, name: str = "SdtHello", control_hz: int = 500, read_only: bool = False):
```
Automatically called by HexDeviceApi to initialize the SdtHello device.

**Parameters:**
- `device_id`: Device ID from SecondaryDeviceStatus
- `device_type`: Device type (SecondaryDeviceType enum, e.g., SdtHello1J1T4BV1)
- `proto_version`: Protocol version as a tuple (major, minor)
- `send_message_callback`: Callback function for sending messages
- `name` (str, optional): Device name, defaults to "SdtHello"
- `control_hz` (int, optional): Control frequency in Hz, defaults to 500
- `read_only` (bool, optional): Whether this device is read-only, defaults to False

**Examples:**
```python
# Usually called internally by HexDeviceApi when creating SdtHello devices
# Users typically don't need to call this directly
```

## has_new_data
```python
def has_new_data(self) -> bool:
```
Checks if there is new Hello device data available.

**Returns:**
- `bool`: True if there is new data, False otherwise

**Examples:**
```python
if hello.has_new_data():
    status = hello.get_simple_motor_status()
    if status is not None:
        print(f"New data available: {status}")
```

## get_simple_motor_status
```python
def get_simple_motor_status(self, pop: bool = True) -> Optional[Dict[str, Any]]:
```
Gets simple Hello device status including joystick, trigger, and button states.

**Parameters:**
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

**Returns:**
- `Optional[Dict[str, Any]]`: Dictionary containing Hello status with keys:
  - `pos`: List of position values [trigger, joystick_x, joystick_y, btn_z, btn_w, btn_x, btn_y]
    - `trigger`: Trigger value (float)
    - `joystick_x`: Joystick X axis value (float)
    - `joystick_y`: Joystick Y axis value (float)
    - `btn_z`, `btn_w`, `btn_x`, `btn_y`: Button states (1.0 for pressed, -1.0 for not pressed)
  - `vel`: List of velocity values (all zeros, not used for Hello device)
  - `eff`: List of effort values (all zeros, not used for Hello device)
  - `ts`: Timestamp dictionary with 's' and 'ns' keys

**Examples:**
```python
status = hello.get_simple_motor_status()
if status is not None:
    trigger = status['pos'][0]
    joystick_x = status['pos'][1]
    joystick_y = status['pos'][2]
    btn_z = status['pos'][3]
    print(f"Trigger: {trigger}, Joystick: ({joystick_x}, {joystick_y}), Button Z: {btn_z}")
```

## set_rgb_stripe_command
```python
def set_rgb_stripe_command(self, r: Union[np.ndarray, list[int]], g: Union[np.ndarray, list[int]], b: Union[np.ndarray, list[int]]):
```
Sets RGB stripe command to control the LED colors on the Hello device.

**Parameters:**
- `r` (Union[np.ndarray, list[int]]): List of red values (0-255) for each LED
- `g` (Union[np.ndarray, list[int]]): List of green values (0-255) for each LED
- `b` (Union[np.ndarray, list[int]]): List of blue values (0-255) for each LED

**Raises:**
- `ValueError`: If the RGB lists have different lengths

**Examples:**
```python
# Set RGB colors for all LEDs
# Example: Set first LED to red, second to green, third to blue
hello.set_rgb_stripe_command(
    r=[255, 0, 0, 0, 0, 0, 0],  # Red values
    g=[0, 255, 0, 0, 0, 0, 0],  # Green values
    b=[0, 0, 255, 0, 0, 0, 0]   # Blue values
)

# Set all LEDs to white
hello.set_rgb_stripe_command(
    r=[255] * 7,
    g=[255] * 7,
    b=[255] * 7
)
```

## get_joint_limits
```python
def get_joint_limits(self) -> List[float]:
```
Gets the joint limits for the Hello device. For Hello devices, these are fixed limits.

**Returns:**
- `List[float]`: List of joint limits for each motor. Each motor's limits are in the format: `[min_pos, max_pos, min_vel, max_vel, min_acc, max_acc]`
  - `min_pos`, `max_pos`: Minimum and maximum position limits (-1.0 to 1.0)
  - `min_vel`, `max_vel`: Velocity limits (0.0, not used)
  - `min_acc`, `max_acc`: Acceleration limits (0.0, not used)

**Examples:**
```python
limits = hello.get_joint_limits()
print(f"Joint limits: {limits}")
```

## get_hello_summary
```python
def get_hello_summary(self) -> dict:
```
Gets comprehensive Hello device summary including device information and configuration.

**Returns:**
- `dict`: Dictionary containing:
  - `name`: Device name
  - `device_id`: Device ID
  - `hello_type`: Device type (SecondaryDeviceType enum value)
  - `control_hz`: Control frequency in Hz

**Examples:**
```python
summary = hello.get_hello_summary()
print(f"Device name: {summary['name']}")
print(f"Device ID: {summary['device_id']}")
print(f"Hello type: {summary['hello_type']}")
print(f"Control frequency: {summary['control_hz']} Hz")
```

## Inherited Methods

The `SdtHello` class inherits all methods from `OptionalDeviceBase` and `MotorBase`:

### From OptionalDeviceBase:
- `get_device_summary()` - Get device status summary (name and device_id)

### From MotorBase:
- `has_new_data()` - Check if new data is available
- `get_motor_position(motor_index)` - Get position of a specific motor
- `get_motor_positions()` - Get positions of all motors
- `get_motor_velocity(motor_index)` - Get velocity of a specific motor
- `get_motor_velocities()` - Get velocities of all motors
- `get_motor_torque(motor_index)` - Get torque of a specific motor
- `get_motor_torques()` - Get torques of all motors
- `get_motor_state(motor_index)` - Get state of a specific motor
- `get_motor_summary()` - Get motor status summary
- `construct_mit_command()` - Construct MIT command from arrays
- `get_motor_error_codes()` - Get motor error codes
- `flush_motor_data()` - Clear motor data queues

**Examples:**
```python
# Get device summary
summary = hello.get_device_summary()
print(f"Device name: {summary['name']}")
print(f"Device ID: {summary['device_id']}")
```

## Usage Example

```python
from hex_device import HexDeviceApi
from hex_device.generated import public_api_types_pb2

# Create API instance
api = HexDeviceApi(ws_url="ws://localhost:8080")

# Find Hello device by device ID
hello = api.find_optional_device_by_id(device_id=5)

# Or find by device type
hello_devices = api.find_optional_device_by_robot_type(robot_type=public_api_types_pb2.SecondaryDeviceType.SdtHello1J1T4BV1)

if hello is not None:
    # Get Hello status
    status = hello.get_simple_motor_status()
    if status is not None:
        trigger = status['pos'][0]
        joystick_x = status['pos'][1]
        joystick_y = status['pos'][2]
        btn_z = status['pos'][3]
        btn_w = status['pos'][4]
        btn_x = status['pos'][5]
        btn_y = status['pos'][6]
        
        print(f"Trigger: {trigger}")
        print(f"Joystick: ({joystick_x}, {joystick_y})")
        print(f"Buttons: Z={btn_z}, W={btn_w}, X={btn_x}, Y={btn_y}")
    
    # Set RGB stripe colors
    # Set first 3 LEDs to red, green, blue respectively
    hello.set_rgb_stripe_command(
        r=[255, 0, 0, 0, 0, 0, 0],
        g=[0, 255, 0, 0, 0, 0, 0],
        b=[0, 0, 255, 0, 0, 0, 0]
    )
    
    # Get Hello summary
    summary = hello.get_hello_summary()
    print(f"Hello type: {summary['hello_type']}")
    print(f"Control frequency: {summary['control_hz']} Hz")
    
    # Get device summary
    device_summary = hello.get_device_summary()
    print(f"Device name: {device_summary['name']}")
    print(f"Device ID: {device_summary['device_id']}")
```
