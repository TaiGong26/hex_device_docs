# ZetaLift API Documentation

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
4. [Calibrate Methods](#calibrate)
   - [calibrate](#calibrate)
   - [is_calibrated](#is_calibrated)
5. [Control Methods](#control-methods)
   - [set_move_speed](#set_move_speed)
6. [State Methods](#state-methods)
   - [get_joint_limits](#get_joint_limits)
   - [get_state](#get_state)
   - [get_my_session_id](#get_my_session_id)
   - [get_parking_stop_detail](#get_parking_stop_detail)
   - [get_status_summary](#get_status_summary)
7. [motor_command](#motor_command)
7. [Inherited Methods](#inherited-methods)
8. [Usage Examples](#usage-examples)



## Overview

The `ZetaLift` class inherits from [DeviceBase](API-Common#DeviceBase) and [MotorBase](API-Motorbase), primarily implementing ZetaLift (rotating lift) status management and motor control. This class processes the `rotate_lift_status` field from APIUp messages.

Supported robot types:
- `RtZetaVc2`: Zeta Vc2 robot type

## Class Definition
```python
class ZetaLift(DeviceBase, MotorBase):
```

The common functions can be found in: [DeviceBase](API-Common#DeviceBase) and [MotorBase](API-Motorbase).

## `__init__`
```python
def __init__(self, motor_count: int, robot_type: int, proto_version: tuple[int, int], name: str = "ZetaLift", control_hz: int = 500, send_message_callback=None):
```
Automatically called by HexDeviceApi to initialize the ZetaLift device.

**Parameters:**
- `motor_count` (int): Number of motors
- `robot_type` (int): Robot type (RobotType enum, e.g., RtZetaVc2)
- `proto_version` (tuple[int, int]): Protocol version as a tuple (major, minor)
- `name` (str, optional): Device name, defaults to "ZetaLift"
- `control_hz` (int, optional): Control frequency in Hz, defaults to 500
- `send_message_callback` (callable, optional): Callback function for sending messages

**Examples:**
```python
# Usually called internally by HexDeviceApi when creating ZetaLift devices
# Users typically don't need to call this directly
```

## calibrate
```python
def calibrate(self):
```
Initiates the calibration process for the ZetaLift. This method sets the calibration flag, which will be processed in the next periodic cycle.

**Examples:**
```python
zeta_lift.calibrate()
# Wait for calibration to complete
while not zeta_lift.is_calibrated():
    await asyncio.sleep(0.1)
```

## set_move_speed
```python
def set_move_speed(self, speed: List[float]):
```
Sets the maximum speed for position mode movement. The speed values are taken as absolute values.

**Parameters:**
- `speed` (List[float]): List of maximum speeds for each motor in position mode

**Examples:**
```python
# Set maximum speed for position mode
zeta_lift.set_move_speed([0.5, 0.5, 0.5])  # Set speed for 3 motors
```

## is_calibrated
```python
def is_calibrated(self) -> bool:
```
Checks if the ZetaLift has been calibrated.

**Returns:**
- `bool`: True if calibrated, False otherwise

**Examples:**
```python
if zeta_lift.is_calibrated():
    print("ZetaLift is calibrated and ready to use")
else:
    print("ZetaLift needs calibration")
```

## get_joint_limits
```python
def get_joint_limits(self) -> Optional[List[List[float]]]:
```
Gets the joint limits for all motors.

**Returns:**
- `Optional[List[List[float]]]`: Joint limits, or None if not available. The list is in the format: `[[min_pos, max_pos], [min_vel, max_vel], [min_acc, max_acc]]`
  - `min_pos`, `max_pos`: Minimum and maximum position limits (pulses)
  - `min_vel`, `max_vel`: Minimum and maximum velocity limits (rad/s)
  - `min_acc`, `max_acc`: Minimum and maximum acceleration limits (rad/s²)

**Examples:**
```python
limits = zeta_lift.get_joint_limits()
if limits is not None:
    for i, motor_limits in enumerate(limits):
        min_pos, max_pos, min_vel, max_vel, min_acc, max_acc = motor_limits
        print(f"Motor {i}: pos=[{min_pos}, {max_pos}], vel=[{min_vel}, {max_vel}], acc=[{min_acc}, {max_acc}]")
```

## get_state
```python
def get_state(self) -> str:
```
Gets the current lift state.

**Returns:**
- `str`: Current lift state name. Possible values:
  - `"LsBrake"`: Brake state
  - `"LsCalibrating"`: Calibrating state
  - `"LsAlgrithmControl"`: Algorithm control state
  - `"LsOvertakeControl"`: Overtake control state
  - `"LsEmergencyStop"`: Emergency stop state

**Examples:**
```python
state = zeta_lift.get_state()
print(f"Current lift state: {state}")
if state == "LsEmergencyStop":
    print("Warning: Lift is in emergency stop state")
```

## get_my_session_id
```python
def get_my_session_id(self) -> int:
```
Gets the session ID associated with this device.

**Returns:**
- `int`: Session ID

**Examples:**
```python
session_id = zeta_lift.get_my_session_id()
print(f"Session ID: {session_id}")
```

## get_parking_stop_detail
```python
def get_parking_stop_detail(self) -> public_api_types_pb2.ParkingStopDetail:
```
Gets the parking stop details, which contain information about why the lift stopped.

**Returns:**
- `ParkingStopDetail`: Parking stop detail object containing:
  - `reason` (str): Reason for parking stop
  - `category` (ParkingStopCategory): Category of parking stop
  - `is_remotely_clearable` (bool): Whether the stop can be cleared remotely

**Examples:**
```python
stop_detail = zeta_lift.get_parking_stop_detail()
if stop_detail.reason:
    print(f"Parking stop reason: {stop_detail.reason}")
    print(f"Category: {stop_detail.category}")
    print(f"Remotely clearable: {stop_detail.is_remotely_clearable}")
```

## get_status_summary
```python
def get_status_summary(self) -> Dict[str, Any]:
```
Gets comprehensive ZetaLift status summary including device information, calibration status, state, position limits, and parking stop details.

**Returns:**
- `Dict[str, Any]`: Dictionary containing:
  - `name`: Device name
  - `calibrated`: Whether the lift is calibrated (bool)
  - `state`: Current lift state (str)
  - `max_pos`: Maximum position limits for each motor (List[int])
  - `min_pos`: Minimum position limits for each motor (List[int])
  - `parking_stop_detail`: Parking stop detail object

**Examples:**
```python
summary = zeta_lift.get_status_summary()
print(f"Device name: {summary['name']}")
print(f"Calibrated: {summary['calibrated']}")
print(f"State: {summary['state']}")
print(f"Position limits: min={summary['min_pos']}, max={summary['max_pos']}")
```

## motor_command
```python
def motor_command(self, command_type: CommandType, values: List[float]):
```
Sets motor command for the ZetaLift. This method extends the base `motor_command` from MotorBase and also records the command timestamp for timeout checking.

**Parameters:**
- `command_type` (CommandType): Command type (BRAKE, SPEED, POSITION, TORQUE, MIT)
- `values`: Command values
  - For BRAKE: List[bool]
  - For SPEED: List[float] (rad/s)
  - For POSITION: List[float] (rad)
  - For TORQUE: List[float] (Nm)
  - For MIT: List[MitMotorCommand]

**Examples:**
```python
from hex_device.motor_base import CommandType

# Set speed command
zeta_lift.motor_command(CommandType.SPEED, [0.5, -0.5, 0.5])

# Set position command
zeta_lift.motor_command(CommandType.POSITION, [0.57, 0.0, 0.1])

```

## Inherited Methods

The `ZetaLift` class inherits methods from both `DeviceBase` and `MotorBase`.

### From DeviceBase:
- `start()` - Start device control
- `stop()` - Stop device control
- `get_device_summary()` - Get device status summary (name)

### From MotorBase:
- `target_positions` - Get all motor target positions (rad)
- `target_velocities` - Get all motor target velocities (rad/s)
- `target_torques` - Get all motor target torques (Nm)
- `has_new_data()` - Check if there is new motor data
- `get_motor_error_codes()` - Get all motor error codes
- `get_motor_state(motor_index)` - Get specified motor state
- `get_motor_states()` - Get all motor states
- `get_simple_motor_status(pop)` - Get simple motor status dictionary
- `get_motor_position(motor_index, pop)` - Get specified motor position (rad)
- `get_motor_positions(pop)` - Get all motor positions (rad)
- `get_motor_encoder_positions(pop)` - Get all motor encoder positions (pulses)
- `get_encoders_to_zero(pop)` - Get encoder values from current position to zero
- `get_motor_velocity(motor_index, pop)` - Get specified motor velocity (rad/s)
- `get_motor_velocities(pop)` - Get all motor velocities (rad/s)
- `get_motor_torque(motor_index, pop)` - Get specified motor torque (Nm)
- `get_motor_torques(pop)` - Get all motor torques (Nm)
- `get_motor_driver_temperatures()` - Get all motor driver temperatures (°C)
- `get_motor_driver_temperature(motor_index)` - Get specified motor driver temperature (°C)
- `get_motor_temperatures()` - Get all motor temperatures (°C)
- `get_motor_temperature(motor_index)` - Get specified motor temperature (°C)
- `get_motor_voltage(motor_index)` - Get specified motor voltage (V)
- `get_motor_voltages()` - Get all motor voltages (V)
- `get_motor_pulse_per_rotation(motor_index)` - Get specified motor pulses per rotation
- `get_motor_pulse_per_rotations()` - Get all motor pulses per rotation
- `get_motor_wheel_radius(motor_index)` - Get specified motor wheel radius (m)
- `get_motor_wheel_radii()` - Get all motor wheel radii (m)
- `mit_motor_command(mit_commands)` - Set MIT motor commands
- `get_motor_summary()` - Get motor group status summary
- `get_motor_status(motor_index, pop)` - Get detailed status for specified motor
- `flush_motor_data()` - Clear all motor data queues

For detailed documentation of these methods, see [MotorBase](API-Motorbase).

**Examples:**
```python
# Start device control
zeta_lift.start()

# Get motor positions
positions = zeta_lift.get_motor_positions()
if positions is not None:
    print(f"Motor positions: {positions}")

# Get motor velocities
velocities = zeta_lift.get_motor_velocities()
if velocities is not None:
    print(f"Motor velocities: {velocities}")

# Get motor summary
motor_summary = zeta_lift.get_motor_summary()
if motor_summary is not None:
    print(f"Motor count: {motor_summary['motor_count']}")
    print(f"States: {motor_summary['states']}")
```

## Usage Examples

### Basic Zeta Lift Control Example
```python
from hex_device import HexDeviceApi
from hex_device.generated import public_api_types_pb2
from hex_device.motor_base import CommandType
import asyncio

# Create API instance
api = HexDeviceApi(ws_url="ws://localhost:8080")

# Find Zeta lift device
zeta_lift = api.find_device_by_robot_type(robot_type=public_api_types_pb2.RobotType.RtZetaVc2)

if zeta_lift is not None:
    # Get current state
    state = zeta_lift.get_state()
    print(f"Current state: {state}")
    
    # Check for parking stop
    stop_detail = zeta_lift.get_parking_stop_detail()
    if stop_detail.reason:
        print(f"Parking stop: {stop_detail.reason}")
    
    # Get joint limits
    limits = zeta_lift.get_joint_limits()
    if limits is not None:
        print(f"Joint limits: {limits}")
    
    # Set move speed 
    zeta_lift.set_move_speed([0.5, 0.5, 0.5])
    
    # Get current motor positions
    positions = zeta_lift.get_motor_positions()
    if positions is not None:
        print(f"Current positions: {positions}")
    
    # Move to target position
    zeta_lift.motor_command(CommandType.POSITION, [1.0, 0.5, 0.0])
    
    # Get status summary
    summary = zeta_lift.get_status_summary()
    print(f"Status summary: {summary}")
    
    # Stop control
    zeta_lift.stop()
```
