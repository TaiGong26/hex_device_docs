# Arm API Documentation
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
5. [Basic Control Methods](#basic-control-methods)
   - [start](#start)
   - [stop](#stop)
   - [command_timeout_check](#command_timeout_check)
   - [motor_command](#motor_command)
   - [clear_parking_stop](#clear_parking_stop)
   - [get_parking_stop_detail](#get_parking_stop_detail)
   - [get_session_holder](#get_session_holder)
   - [get_my_session_id](#get_my_session_id)
   - [enable_mit](#enable_mit)
6. [Configuration Methods](#configuration-methods)
   - [get_arm_config](#get_arm_config)
   - [get_joint_limits](#get_joint_limits)
   - [get_joint_names](#get_joint_names)
   - [get_expected_motor_count](#get_expected_motor_count)
   - [check_motor_count_match](#check_motor_count_match)
   - [get_arm_series](#get_arm_series)
   - [get_arm_name](#get_arm_name)
7. [Advanced Control Methods](#advanced-control-methods)
   <!-- - [end_effector_control](#end_effector_control)
   - [enable_free_drag](#enable_free_drag)
   - [enable_zero_current_control](#enable_zero_current_control)
   - [compensated_mit_control](#compensated_mit_control) -->
8. [Validation Methods](#validation-methods)
   - [validate_joint_positions](#validate_joint_positions)
   - [validate_joint_velocities](#validate_joint_velocities)
   - [is_timeout](#is_timeout)
9. [Configuration Management](#configuration-management)
   - [reload_arm_config_from_dict](#reload_arm_config_from_dict)
10. [Motion History Management](#motion-history-management)
    - [get_last_positions](#get_last_positions)
    - [get_last_velocities](#get_last_velocities)
    - [clear_position_history](#clear_position_history)
    - [clear_velocity_history](#clear_velocity_history)
    - [clear_motion_history](#clear_motion_history)


## Overview

The `Arm` class inherits from [DeviceBase](API-Common.md#DeviceBase) and [MotorBase](API-Motorbase.md), primarily implementing the control of robotic arm devices. This class corresponds to `ArmStatus` in the proto, managing arm status and motor control.

## Supported Robot Types

| Robot Type | Degrees of Freedom | Model | ID |
|------------|-------------------|-------|----|
| `RtArmSaberD6x` | 6 | Saber | 14 |
| `RtArmSaberD7x` | 7 | Saber | 15 |
| `RtArmArcherD6Y_P1` | 6 | Archer | 16 |
| `RtArmArcherY6L_V1` | 6 | Archer | 17 |
| `RtArmArcherY6_H1` | 6 | Archer | 25 |
| `RtArmFireflyY6_H1` | 6 | Firefly | 27 |
| `RtHelloArcherY6_H1` | 6 | Hello Archer | 26 |
| `RtHelloFireflyY6_H1` | 6 | Hello Firefly | 28 |
| `RtArmArcherX7h1` | 7 | Archer | 29 |

# Arm
```python
class Arm(DeviceBase, MotorBase):
```

The common function can be found in: [DeviceBase](API-Common.md#DeviceBase) and [MotorBase](API-Motorbase.md).

## `__init__`
```python
def __init__(self, robot_type, motor_count, proto_version, name: str = "Arm", control_hz: int = 500, send_message_callback=None):
```
Automatically called by HexDeviceApi to initialize the Arm robotic arm device.

**Parameters:**
- `robot_type`: Robot type (RobotType enum or int ID)
- `motor_count` (int): Number of motors
- `proto_version` (tuple[int, int]): Protocol version as a tuple (major, minor)
- `name` (str, optional): Device name, defaults to "Arm"
- `control_hz` (int, optional): Control frequency in Hz, defaults to 500
- `send_message_callback` (callable, optional): Callback function for sending messages

**Examples:**
```python
# Usually called internally by HexDeviceApi
arm = Arm(robot_type=16, motor_count=6, proto_version=(1, 2), name="MyArm", control_hz=500)
```

## `start`
```python
def start(self):
```
Sends initialization command and sets `api_control_initialized` to `True`. The robotic arm will only start responding to control commands after calling this function.  
**Note:** If there is already a controller, this command will be ignored by the robotic arm until control is released.

```python
api = HexDeviceApi(ws_url=args.url, control_hz=250)
arm = api.find_device_by_robot_type(16)
arm.start()
```

## `stop`
```python
def stop(self):
```
Sends stop command and sets `api_control_initialized` to `False`. After calling this function, the robotic arm will enter Disable mode and stop connection monitoring and command listening. This is the correct way to disconnect. If you exit the program directly without calling stop, the robotic arm will enter a connection timeout error state.

```python
api = HexDeviceApi(ws_url=args.url, control_hz=250)
arm = api.find_device_by_robot_type(16)
arm.stop()
```

## command_timeout_check
```python
def command_timeout_check(self, check_or_not: bool = True):
```
Sets whether to check for command timeout. When enabled, the arm will automatically brake if no commands are received within the timeout period (default 300ms).

Examples:
```python
# Enable command timeout checking (default)
arm.command_timeout_check(True)

# Disable command timeout checking
arm.command_timeout_check(False)
```

## motor_command
```python
def motor_command(self, command_type: CommandType, values: Union[List[bool], List[float], List[MitMotorCommand], np.ndarray]):
```
> **Note:** For detailed command types and usage, see [motor_command](API-Motorbase.md#motor_command) in API-Motorbase.

Sets robotic arm motor commands with automatic validation for position and velocity commands.  
> **Warning!!!!**Only one command can be set at the same time.

**Parameters:**
- `command_type` (CommandType): Command type (BRAKE, SPEED, POSITION, TORQUE, MIT)
- `values`: Command values
  - For BRAKE: List[bool] (length must match motor count)
  - For SPEED: List[float] (rad/s) - automatically validated
  - For POSITION: List[float] (rad) - automatically validated
  - For TORQUE: List[float] (Nm)
  - For MIT: List[MitMotorCommand]

**Note:**
- Only POSITION and SPEED commands are automatically validated against joint limits
- MIT and TORQUE commands may not be enabled by default on certain arm types (e.g., Saber arms). Use `enable_mit()` to enable if needed.
- Raises `ValueError` if MIT/TORQUE commands are used on arms that don't support them

**Examples:**
```python
from hex_device.motor_base import CommandType

# Set joint position commands (with automatic validation)
arm.motor_command(CommandType.POSITION, [0.0, 0.5, 1.0, 0.0, 0.5, 0.0])

# Set joint velocity commands (with automatic validation)
arm.motor_command(CommandType.SPEED, [0.1, -0.1, 0.2, 0.0, -0.1, 0.0])

# Set brake commands
arm.motor_command(CommandType.BRAKE, [True] * 6)

# Set torque commands (may require enable_mit() first)
arm.motor_command(CommandType.TORQUE, [0.5, 0.3, 0.2, 0.1, 0.1, 0.0])

# Set MIT commands (may require enable_mit() first)
mit_commands = arm.construct_mit_command(
    pos=[0.0, 0.5, 1.0, 0.0, 0.5, 0.0],  # Target positions
    speed=[0.0, 0.0, 0.0, 0.0, 0.0, 0.0],  # Target speeds
    torque=[0.0, 0.0, 0.0, 0.0, 0.0, 0.0],  # Target torques
    kp=[100.0] * 6,  # Proportional gains
    kd=[10.0] * 6    # Derivative gains
)
arm.motor_command(CommandType.MIT, mit_commands)
```

## clear_parking_stop
```python
def clear_parking_stop(self):
```
Clears the parking stop that can be cleared remotely. This method sets a flag that will be processed in the next periodic cycle.

**Examples:**
```python
# Clear parking stop if it's remotely clearable
stop_detail = arm.get_parking_stop_detail()
if stop_detail.is_remotely_clearable:
    arm.clear_parking_stop()
```

## get_parking_stop_detail
```python
def get_parking_stop_detail(self) -> public_api_types_pb2.ParkingStopDetail:
```
Gets the detailed reason for arm emergency stop.

Examples:
```python
stop_detail = arm.get_parking_stop_detail()
if stop_detail.category != public_api_types_pb2.ParkingStopCategory.PscNone:
    print(f"Emergency stop reason: {stop_detail.reason}")
    print(f"Category: {stop_detail.category}")
```

## get_session_holder
```python
def get_session_holder(self) -> int:
```
Gets the session ID of the current controller. Returns 0 when no one is controlling the robotic arm.

**Returns:**
- `int`: Session ID of the current controller, or 0 if no one is controlling

**Examples:**
```python
session_id = arm.get_session_holder()
if session_id == 0:
    print("No one is controlling, you can use start() to try to get control")
elif session_id == arm.get_my_session_id():
    print("You are currently controlling the arm")
else:
    print(f"Another controller (session ID: {session_id}) is controlling the arm")
```

## get_my_session_id
```python
def get_my_session_id(self) -> int:
```
Gets the session ID of the current connection. This ID is assigned by the server.

**Returns:**
- `int`: Session ID of the current connection

**Examples:**
```python
my_id = arm.get_my_session_id()
holder_id = arm.get_session_holder()
print(f"My session ID: {my_id}")
if my_id == holder_id:
    print("I am controlling the arm")
```

## enable_mit
```python
def enable_mit(self):
```
Enables MIT command support for the robotic arm. By default, MIT commands are disabled on Saber arms (ID: 14, 15) for safety reasons.

**Note:** This is a soft lock for Saber arms. Enabling MIT commands on unsupported arms may cause errors.

**Examples:**
```python
# Enable MIT commands (required for Saber arms before using MIT/TORQUE commands)
arm.enable_mit()

# Now MIT and TORQUE commands can be used
arm.motor_command(CommandType.TORQUE, [0.5, 0.3, 0.2, 0.1, 0.1, 0.0])
```

## Configuration Methods

### get_arm_config
```python
def get_arm_config(self) -> Optional[ArmConfig]:
```
Gets the current robotic arm configuration.

Examples:
```python
config = arm.get_arm_config()
if config:
    print(f"Arm name: {config.name}")
    print(f"Motor robot_config: {config.robot_config}")
```

### get_joint_limits
```python
def get_joint_limits(self) -> Optional[List[List[float]]]:
```
Gets joint limits for each joint. The limits include position, velocity, and acceleration constraints.

**Returns:**
- `Optional[List[List[float]]]`: List of joint limits for each joint, or None if not available. Each joint's limits are in the format: `[min_pos, max_pos, min_vel, max_vel, min_acc, max_acc]`
  - `min_pos`, `max_pos`: Minimum and maximum position limits (rad)
  - `min_vel`, `max_vel`: Minimum and maximum velocity limits (rad/s)
  - `min_acc`, `max_acc`: Minimum and maximum acceleration limits (rad/s²)

**Examples:**
```python
limits = arm.get_joint_limits()
if limits:
    for i, joint_limits in enumerate(limits):
        min_pos, max_pos, min_vel, max_vel, min_acc, max_acc = joint_limits
        print(f"Joint {i}: pos=[{min_pos:.3f}, {max_pos:.3f}], vel=[{min_vel:.3f}, {max_vel:.3f}], acc=[{min_acc:.3f}, {max_acc:.3f}]")
```

### get_joint_names
```python
def get_joint_names(self) -> Optional[List[str]]:
```
Gets the names of all joints.

Examples:
```python
joint_names = arm.get_joint_names()
if joint_names:
    print(f"Joint names: {joint_names}")
```

### get_expected_motor_count
```python
def get_expected_motor_count(self) -> Optional[int]:
```
Gets the expected motor count for this arm series.

Examples:
```python
expected_count = arm.get_expected_motor_count()
actual_count = arm.motor_count
if expected_count != actual_count:
    print(f"Motor count mismatch: expected {expected_count}, actual {actual_count}")
```

### check_motor_count_match
```python
def check_motor_count_match(self) -> bool:
```
Checks if the actual motor count matches the configuration.

Examples:
```python
if not arm.check_motor_count_match():
    print("Warning: Motor count does not match configuration")
```

### get_arm_series
```python
def get_arm_series(self) -> int:
```
Gets the arm series ID.

Examples:
```python
series_id = arm.get_arm_series()
print(f"Arm series ID: {series_id}")
```

### get_arm_name
```python
def get_arm_name(self) -> Optional[str]:
```
Gets the arm name from configuration.

Examples:
```python
arm_name = arm.get_arm_name()
if arm_name:
    print(f"Arm name: {arm_name}")
```

## Advanced Control Methods

> **Note:** This method is under development
<!-- 
### end_effector_control
```python
def end_effector_control(self, position: Union[List[float], Tuple[float, float, float]], orientation: Union[List[float], Tuple[float, float, float, float], None] = None, gravity_acc: Optional[List[float]] = None):
```
Controls the position and orientation of the arm's end effector.

**Parameters:**
- `position` (Union[List[float], Tuple[float, float, float]]): End effector position (x, y, z) in meters, length must be 3
- `orientation` (Union[List[float], Tuple[float, float, float, float], None], optional): End effector orientation as quaternion (qw, qx, qy, qz), defaults to [1, 0, 0, 0] (identity quaternion)
- `gravity_acc` (Optional[List[float]], optional): Gravity acceleration (ax, ay, az) for compensation, length must be 3 when provided

**Notes:**
- This method allows direct control of the end effector position and orientation without manually calculating joint angles
- Suitable for scenarios requiring precise end effector positioning
- Gravity compensation improves control accuracy, especially for vertical movements

Examples:
```python
# Control end effector to specified position (default orientation)
arm.end_effector_control(
    position=[0.5, 0.0, 0.5]  # x=0.5m, y=0.0m, z=0.5m
)

# Control end effector to specified position and orientation
arm.end_effector_control(
    position=[0.5, 0.0, 0.5],
    orientation=[1.0, 0.0, 0.0, 0.0]  # Identity quaternion (default orientation)
)

# End effector control with gravity compensation
arm.end_effector_control(
    position=[0.5, 0.0, 0.5],
    gravity_acc=[0.0, 0.0, 9.81]  # Gravity acceleration
)
```

### joint_position_control
```python
def joint_position_control(self, joint_positions: List[float], velocities: List[float], accelerations: List[float]):
```
A more convenient joint position control mode that plans the target position before moving.

**Parameters:**
- `joint_positions` (List[float]): Joint positions in radians, length must match the motor count
- `velocities` (List[float]): Joint velocities in rad/s, length must match the motor count
- `accelerations` (List[float]): Joint accelerations in rad/s², length must match the motor count

**Notes:**
- This method automatically plans the joint movement path to smoothly move the arm to the target position
- Suitable for scenarios requiring precise joint angle control
- Internal joint limit validation is performed to ensure safe movement
- The method uses the arm's built-in trajectory planning capabilities

**Examples:**
```python
# Control arm to specified joint positions with velocities and accelerations
arm.joint_position_control(
    joint_positions=[0.0, 0.5, 1.0, 0.0, 0.5, 0.0],  # Target positions for 6 joints
    velocities=[0.5, 0.5, 0.5, 0.5, 0.5, 0.5],        # Target velocities for 6 joints
    accelerations=[1.0, 1.0, 1.0, 1.0, 1.0, 1.0]      # Target accelerations for 6 joints
)

# Get current joint positions and then make fine adjustments
current_positions = arm.get_motor_positions()
if current_positions is not None:
    # Make fine adjustments based on current positions
    new_positions = [pos + 0.1 for pos in current_positions]
    arm.joint_position_control(new_positions)
```

### enable_free_drag
```python
def enable_free_drag(self, gravity_acc: list[float]):
```
Enables free drag mode, allowing manual manipulation of the arm.

**Parameters:**
- `gravity_acc` (list[float]): Gravity acceleration (ax, ay, az) for compensation, length must be 3

**Notes:**
- In free drag mode, the arm enters a low-impedance state, allowing the user to manually move it
- Gravity compensation parameters are used to counteract the effects of gravity, making dragging easier
- Suitable for teaching, programming demonstration, and manual arm position adjustment
- While in free drag mode, the arm will not execute other control commands
- To exit free drag mode, send another control command such as position or speed

Examples:
```python
# Enable free drag mode (with gravity compensation)
arm.enable_free_drag(
    gravity_acc=[0.0, 0.0, 9.81]  # Gravity acceleration
)
print("Free drag mode enabled. You can now manually move the arm.")

# Note: While in free drag mode, the arm will not execute other control commands
# To exit free drag mode, send another control command, such as position or speed
```

### enable_zero_current_control
```python
def enable_zero_current_control(self):
```
Enables zero current control mode for the arm.

**Notes:**
- Zero current control mode disables torque output, allowing the arm to move freely
- Use this mode when you want to manually move the arm without any resistance
- Be cautious when using this mode as the arm will not maintain its position

Examples:
```python
# Enable zero current control mode
arm.enable_zero_current_control()
print("Zero current control mode enabled. Arm can now be moved freely.")

# Note: In zero current mode, the arm will not hold its position
# Use gravity compensation if you need to maintain position while allowing movement
```

### compensated_mit_control
```python
def compensated_mit_control(self, mit_commands: List[MitMotorCommand], gravity_acc: List[float]):
```
Compensated MIT control mode with higher precision.

**Parameters:**
- `mit_commands` (List[MitMotorCommand]): MIT command list, each element contains torque, speed, position, kp, and kd parameters
- `gravity_acc` (List[float]): Gravity acceleration (ax, ay, az) for compensation, length must be 3

**Notes:**
- MIT control allows simultaneous control of joint position, speed, and torque
- Gravity compensation improves control accuracy, especially for vertical movements
- Suitable for high-precision control scenarios such as fine manipulation and force control
- Each MitMotorCommand includes:
  - `torque`: Target torque (Nm)
  - `speed`: Target speed (rad/s)
  - `position`: Target position (rad)
  - `kp`: Proportional gain
  - `kd`: Derivative gain

Examples:
```python
# Construct MIT commands
mit_commands = arm.construct_mit_command(
    pos=[0.0, 0.5, 1.0, 0.0, 0.5, 0.0],  # Target positions
    speed=[0.0, 0.0, 0.0, 0.0, 0.0, 0.0],  # Target speeds
    torque=[0.0, 0.0, 0.0, 0.0, 0.0, 0.0],  # Target torques
    kp=[100.0] * 6,  # Proportional gains
    kd=[10.0] * 6     # Derivative gains
)

# Compensated MIT control with gravity compensation
arm.compensated_mit_control(
    mit_commands=mit_commands,
    gravity_acc=[0.0, 0.0, 9.81]  # Gravity acceleration
)
```
 -->
## Validation Methods

### validate_joint_positions
```python
def validate_joint_positions(self, positions: List[float], dt: float = 0.002) -> List[float]:
```
Validates and clamps joint positions to be within limits, considering velocity constraints.

Examples:
```python
target_positions = [0.0, 1.5, 2.0, 0.0, 1.0, 0.0]
safe_positions = arm.validate_joint_positions(target_positions, dt=0.002)
print(f"Original: {target_positions}")
print(f"Validated: {safe_positions}")
```

### validate_joint_velocities
```python
def validate_joint_velocities(self, velocities: List[float], dt: float = 0.002) -> List[float]:
```
Validates and clamps joint velocities to be within limits.

Examples:
```python
target_velocities = [1.0, -2.0, 3.0, 0.5, -1.5, 0.8]
safe_velocities = arm.validate_joint_velocities(target_velocities, dt=0.002)
print(f"Original: {target_velocities}")
print(f"Validated: {safe_velocities}")
```

### is_timeout
```python
def is_timeout(self) -> bool:
```
Checks if the arm control has timed out.

**Returns:**
- `bool`: True if control has timed out, False otherwise

Examples:
```python
if arm.is_timeout():
    print("Arm control has timed out")
else:
    print("Arm control is active")
```

## Configuration Management

### reload_arm_config_from_dict
```python
def reload_arm_config_from_dict(self, config_data: dict) -> bool:
```
Reloads arm configuration from dictionary data. Used to update joint limits and validation parameters.

Examples:
```python
config_data = {
    "name": "CustomArm",
    "motor_count": 6,
    "joint_limits": [[-3.14, 3.14]] * 6,
    "max_joint_velocities": [2.0] * 6,
    "max_joint_accelerations": [5.0] * 6
}

success = arm.reload_arm_config_from_dict(config_data)
if success:
    print("Configuration reloaded successfully")
else:
    print("Failed to reload configuration")
```

## Motion History Management
<!-- 
### set_initial_positions
```python
def set_initial_positions(self, positions: List[float]):
```
Sets the initial positions for velocity limit calculations.

Examples:
```python
initial_pos = [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
arm.set_initial_positions(initial_pos)
```

### set_initial_velocities
```python
def set_initial_velocities(self, velocities: List[float]):
```
Sets the initial velocities for acceleration limit calculations.

Examples:
```python
initial_vel = [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
arm.set_initial_velocities(initial_vel)
```
 -->
### get_last_positions
```python
def get_last_positions(self) -> Optional[List[float]]:
```
Gets the last recorded joint positions.

Examples:
```python
last_pos = arm.get_last_positions()
if last_pos:
    print(f"Last positions: {last_pos}")
```

### get_last_velocities
```python
def get_last_velocities(self) -> Optional[List[float]]:
```
Gets the last recorded joint velocities.

Examples:
```python
last_vel = arm.get_last_velocities()
if last_vel:
    print(f"Last velocities: {last_vel}")
```

### clear_position_history
```python
def clear_position_history(self):
```
Clears the position history records.

Examples:
```python
arm.clear_position_history()
print("Position history cleared")
```

### clear_velocity_history
```python
def clear_velocity_history(self):
```
Clears the velocity history records.

Examples:
```python
arm.clear_velocity_history()
print("Velocity history cleared")
```

### clear_motion_history
```python
def clear_motion_history(self):
```
Clears all motion history records (both position and velocity).

**Examples:**
```python
arm.clear_motion_history()
print("All motion history cleared")
```

## Inherited Methods

The `Arm` class inherits methods from both `DeviceBase` and `MotorBase`.

### From DeviceBase:
- `start()` - Start device control (send initialization command)
- `stop()` - Stop device control (send stop command)
- `get_device_summary()` - Get device status summary (name)
- `get_status_summary()` - Get complete device status summary including arm-specific fields

### From MotorBase:
- `target_positions` - Get all motor target positions (rad)
- `target_velocities` - Get all motor target velocities (rad/s)
- `target_torques` - Get all motor target torques (Nm)
- `cache_motion_data` - Get all motor cache motion data (positions, velocities, torques)
- `cache_positions` - Get all motor cache positions (rad)
- `cache_velocities` - Get all motor cache velocities (rad/s)
- `cache_torques` - Get all motor cache torques (Nm)
- `has_new_data()` - Check if there is new motor data
- `get_motor_warnings()` - Get all motor warnings
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
- `construct_mit_command()` - Constructs MIT command from numpy array or list
- `get_motor_summary()` - Get motor group status summary
- `get_motor_status(motor_index, pop)` - Get detailed status for specified motor
- `flush_motor_data()` - Clear all motor data queues

For detailed documentation of these methods, see [MotorBase](API-Motorbase.md).

**Examples:**
```python
# Get motor positions
positions = arm.get_motor_positions()
if positions is not None:
    print(f"Current joint positions: {positions}")

# Get motor velocities
velocities = arm.get_motor_velocities()
if velocities is not None:
    print(f"Current joint velocities: {velocities}")

# Get motor summary
motor_summary = arm.get_motor_summary()
if motor_summary is not None:
    print(f"Motor count: {motor_summary['motor_count']}")
    print(f"States: {motor_summary['states']}")
```

## Usage Example

```python
from hex_device import HexDeviceApi
from hex_device.generated import public_api_types_pb2
from hex_device.motor_base import CommandType
import asyncio

# Create API instance
api = HexDeviceApi(ws_url="ws://localhost:8080")

# Find arm device by robot type
arm = api.find_device_by_robot_type(robot_type=public_api_types_pb2.RobotType.RtArmArcherD6Y_P1)

if arm is not None:
    # Start device control
    arm.start()
    
    # Wait until we get control
    while arm.get_session_holder() != arm.get_my_session_id():
        await asyncio.sleep(0.1)
    print("Got control of the arm")
    
    # Check calibration status
    config = arm.get_arm_config()
    if config:
        print(f"Arm name: {config.name}")
        print(f"Expected motor count: {arm.get_expected_motor_count()}")
    
    # Get joint limits
    limits = arm.get_joint_limits()
    if limits:
        print(f"Joint limits: {limits}")
    
    # Get current positions
    positions = arm.get_motor_positions()
    if positions is not None:
        print(f"Current positions: {positions}")
    
    # Move to target position (with automatic validation)
    arm.motor_command(CommandType.POSITION, [0.0, 0.5, 1.0, 0.0, 0.5, 0.0])
    
    # Check for parking stop
    stop_detail = arm.get_parking_stop_detail()
    if stop_detail.reason:
        print(f"Parking stop: {stop_detail.reason}")
        if stop_detail.is_remotely_clearable:
            arm.clear_parking_stop()
    
    # Stop device control
    arm.stop()
```
<!-- 
## Best Practices

1. **Always call `stop()` before exiting**
   - Ensures proper disconnection and prevents timeout errors

2. **Use automatic validation**
   - Take advantage of automatic position and velocity validation
   - Prevents sending invalid commands to the arm

3. **Check session holder before starting**
   - Use `get_session_holder()` to check if another controller is active

4. **Monitor parking stop status**
   - Regularly check `get_parking_stop_detail()` to detect issues

5. **Use MIT commands for precise control**
   - MIT commands offer more precise control with PID gains
   - Enable MIT mode with `enable_mit()` when needed

## Troubleshooting

1. **Cannot Start Control**
   - **Symptom**: `start()` has no effect
   - **Solution**: Wait for other controller to release control or restart arm

2. **Motor Count Mismatch**
   - **Symptom**: `check_motor_count_match()` returns False
   - **Solution**: Verify robot type and check arm configuration

3. **Parking Stop**
   - **Symptom**: Arm stops unexpectedly
   - **Solution**: Check `get_parking_stop_detail()` for reason and clear if possible

4. **Command Timeout**
   - **Symptom**: Arm enters timeout state
   - **Solution**: Increase command sending frequency

5. **Debugging Tips**
   - Enable verbose logging to see detailed communication
   - Regularly check `get_arm_config()` and `get_arm_series()`
   - Test with simple commands before complex operations
 -->