# LinearLift API Documentation
<!-- 
## Version Information

- **API Version**: 1.0
- **Protocol Version**: (1, 0)
- **Compatibility**: Requires HexDevice Python SDK v1.0 or later
 -->

## Table of Contents

1. [Overview](#overview)
2. [Supported Robot Types](#supported-robot-types)
3. [Class Definition](#class-definition)
4. [Initialization](#__init__)
5. [Data Methods](#data-methods)
   - [has_new_data](#has_new_data)
   - [get_parking_stop_detail](#get_parking_stop_detail)
   - [get_state](#get_state)
   - [get_pos_range](#get_pos_range)
   - [get_motor_positions](#get_motor_positions)
   - [get_move_speed](#get_move_speed)
   - [get_max_move_speed](#get_max_move_speed)
   - [get_pulse_per_meter](#get_pulse_per_meter)
6. [Control Methods](#control-methods)
   - [motor_command](#motor_command)
   - [set_move_speed](#set_move_speed)
   - [calibrate](#calibrate)
7. [Inherited Methods](#inherited-methods)
8. [Usage Examples](#usage-examples)

## Overview

The `LinearLift` class inherits from [DeviceBase](API-Common#Devicebase), primarily implementing linear lift control and status management. This class processes the `linear_lift_status` field from APIUp messages.

## Supported Robot Types
- `RtIotaP1`: Iota P1 Linear Lift
- `RtIotaVc1`: Iota Vc1 Linear Lift

## Class Definition

```python
class LinearLift(DeviceBase):
```

Common functions can be found in: [DeviceBase](API-Common#Devicebase)


## `__init__`
```python
def __init__(self, motor_count: int, robot_type: int, name: str = "Lift", control_hz: int = 500, send_message_callback=None):
```
Initializes a LinearLift device for linear lift control.

**Parameters:**
- `motor_count` (int): Number of motors in the lift
- `robot_type` (int): Robot type (RobotType enum)
- `name` (str, optional): Device name, defaults to "Lift"
- `control_hz` (int, optional): Control frequency in Hz, defaults to 500
- `send_message_callback` (callable, optional): Callback function for sending messages

**Examples:**
```python
# Usually called internally by HexDeviceApi when creating lift devices
# Users typically don't need to call this directly
```

## has_new_data
```python
def has_new_data(self) -> bool:
```
Checks if there is lift data available.

**Returns:**
- `bool`: True if there is data coming, False otherwise

**Examples:**
```python
if lift.has_new_data():
    # Get current lift state
    position = lift.get_motor_positions()
    speed = lift.get_move_speed()
```

## get_parking_stop_detail
```python
def get_parking_stop_detail(self) -> Optional[public_api_types_pb2.ParkingStopDetail]:
```
Gets the parking stop detail information. Returns `None` if there is no parking stop.

**Returns:**
- `Optional[ParkingStopDetail]`: Parking stop detail or None if no parking stop

**Examples:**
```python
parking_detail = lift.get_parking_stop_detail()
if parking_detail is not None:
    print(f"Parking stop detected: {parking_detail}")
```

## get_state
```python
def get_state(self) -> str:
```
Gets the current lift state.

**Returns:**
- `str`: Lift state name (e.g., "LsBrake", "LsCalibrating", "LsAlgrithmControl", "LsOvertakeControl", "LsEmergencyStop")

**Examples:**
```python
state = lift.get_state()
print(f"Lift state: {state}")
if state == "LsBrake":
    print("Lift is in brake mode")
```

## get_pos_range
```python
def get_pos_range(self) -> Tuple[float, float]:
```
Gets the position range of the lift in meters.

**Returns:**
- `Tuple[float, float]`: Position range (min, max) in meters. Returns (0, 0) if not calibrated.

**Examples:**
```python
min_pos, max_pos = lift.get_pos_range()
print(f"Position range: {min_pos} to {max_pos} meters")
```

## get_motor_positions
```python
def get_motor_positions(self) -> List[float]:
```
Gets the current motor positions in meters.

**Returns:**
- `List[float]`: Current motor positions (m)

**Examples:**
```python
positions = lift.get_motor_positions()
print(f"Current position: {positions[0]} meters")
```

## get_move_speed
```python
def get_move_speed(self) -> float:
```
Gets the current move speed.

**Returns:**
- `float`: Current move speed (pulse/s)

**Examples:**
```python
speed = lift.get_move_speed()
print(f"Current speed: {speed} pulse/s")
```

## get_max_move_speed
```python
def get_max_move_speed(self) -> float:
```
Gets the maximum move speed that can be set.

**Returns:**
- `float`: Maximum move speed (pulse/s)

**Examples:**
```python
max_speed = lift.get_max_move_speed()
print(f"Max speed: {max_speed} pulse/s")
```

## get_pulse_per_meter
```python
def get_pulse_per_meter(self) -> float:
```
Gets the pulse per meter conversion factor.

**Returns:**
- `float`: Pulse per meter

**Examples:**
```python
ppr = lift.get_pulse_per_meter()
print(f"Pulse per meter: {ppr}")
```

## motor_command
```python
def motor_command(self, command_type: CommandType, values: Union[bool, float, np.ndarray]):
```
Set motor command for the lift. Supports POSITION and BRAKE command types.

**Parameters:**
- `command_type`: Command type (POSITION or BRAKE)
- `values`: Command values
  - For POSITION: Position value in meters (float or numpy array)
  - For BRAKE: Boolean value (True to apply brake)

**Notes:**
- Position commands are automatically clamped to valid range [0, max_pos]
- Position values are automatically converted from meters to encoder positions
- The lift must be calibrated before sending position commands

**Examples:**
```python
# Position command (in meters)
lift.motor_command(CommandType.POSITION, 0.5)

# Using numpy arrays
import numpy as np
position = np.array([0.5])
lift.motor_command(CommandType.POSITION, position)

# Brake command
lift.motor_command(CommandType.BRAKE, True)
```

## set_move_speed
```python
def set_move_speed(self, speed: Union[float, np.ndarray]):
```
Set the move speed for the lift.

**Parameters:**
- `speed` (float or np.ndarray): Target speed (pulse/s). Automatically clamped to [0, max_speed]

**Examples:**
```python
# Set move speed
lift.set_move_speed(1000)  # 1000 pulse/s

# Speed will be clamped to max_speed if exceeded
lift.set_move_speed(5000)  # Will be clamped to max_speed
```

## calibrate
```python
def calibrate(self):
```
Calibrate the lift. The lift must be calibrated before moving when powered on.

**Warning:**
- It is strictly forbidden to send the calibrate command continuously!
- Only send calibrate command when necessary (e.g., after power on or after clearing parking stop)

**Examples:**
```python
# Calibrate the lift (usually done once after power on)
lift.calibrate()

# Wait for calibration to complete
import time
while lift.get_state() == "LsCalibrating":
    time.sleep(0.1)
```

## Inherited Methods

The `LinearLift` class inherits all methods from `DeviceBase`, including:

### From DeviceBase:
- `start()` - Start device control
- `stop()` - Stop device control
- `get_device_summary()` - Get device status summary

**Examples:**
```python
# Start lift control
lift.start()

# Check device status
summary = lift.get_device_summary()
print(f"Device name: {summary['name']}")

# Stop lift control
lift.stop()
```

## Usage Examples

### Basic Lift Control Example
```python
from hex_device import HexDeviceApi
from hex_device.generated import public_api_types_pb2
from hex_device.motor_base import CommandType

# Create API instance
api = HexDeviceApi(ws_url="ws://localhost:8080")

# Find lift device
lift = api.find_device_by_robot_type(robot_type=public_api_types_pb2.RobotType.RtIotaP1)

if lift is not None:
    # Start control
    lift.start()
    print("Lift control started")
    
    # Calibrate lift
    print("Calibrating lift...")
    lift.calibrate()
    
    # Wait for calibration to complete
    import time
    while lift.get_state() == "LsCalibrating":
        time.sleep(0.1)
    
    # Set move speed
    lift.set_move_speed(1000)  # 1000 pulse/s
    print("Move speed set to 1000 pulse/s")
    
    # Move to target position (0.5 meters)
    print("Moving to 0.5 meters...")
    lift.motor_command(CommandType.POSITION, 0.5)
    
    # Wait for a while
    time.sleep(2)
    
    # Check current position
    if lift.has_new_data():
        position = lift.get_motor_positions()
        print(f"Current position: {position[0]} meters")
    
    # Apply brake
    lift.motor_command(CommandType.BRAKE, True)
    
    # Stop control
    lift.stop()
```

