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
5. [Configuration Management](#configuration-management)
   - [reload_arm_config_from_dict](#reload_arm_config_from_dict)
6. [Basic Control Methods](#basic-control-methods)
   - [start](#start)
   - [stop](#stop)
   - [command_timeout_check](#command_timeout_check)
   - [motor_command](#motor_command)
   - [clear_parking_stop](#clear_parking_stop)
   - [enable_mit](#enable_mit)
7. [Advanced Control Methods](#advanced-control-methods)
   <!-- - [end_effector_control](#end_effector_control)
   - [enable_free_drag](#enable_free_drag)
   - [enable_zero_current_control](#enable_zero_current_control)
   - [compensated_mit_control](#compensated_mit_control) -->
8. [Configuration Methods](#configuration-methods)
   - [get_arm_config](#get_arm_config)
   - [get_joint_limits](#get_joint_limits)
   - [get_joint_names](#get_joint_names)
   - [get_expected_motor_count](#get_expected_motor_count)
   - [check_motor_count_match](#check_motor_count_match)
   - [get_arm_series](#get_arm_series)
   - [get_arm_name](#get_arm_name)
   - [get_parking_stop_detail](#get_parking_stop_detail)
   - [get_session_holder](#get_session_holder)
   - [get_my_session_id](#get_my_session_id)
9. [Validation Methods](#validation-methods)
   - [validate_joint_positions](#validate_joint_positions)
   - [validate_joint_velocities](#validate_joint_velocities)
   - [is_timeout](#is_timeout)
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

## Class Definition

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

## Configuration Management

### reload_arm_config_from_dict

```python
def reload_arm_config_from_dict(self, config_data: dict) -> bool:
```

Reloads arm configuration from dictionary data. Used to update joint limits and validation parameters.

**Examples:**

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

## Basic Control Methods

### start

```python
def start(self):
```

Sends initialization command and sets `api_control_initialized` to `True`. The robotic arm will only start responding to control commands after calling this function.
**Note:** If there is already a controller, this command will be ignored by the robotic arm until control is released.

**Examples:**

```python
api = HexDeviceApi(ws_url=args.url, control_hz=250)
arm = api.find_device_by_robot_type(16)
arm.start()
```

### stop

```python
def stop(self):
```

Sends stop command and sets `api_control_initialized` to `False`. After calling this function, the robotic arm will enter Disable mode and stop connection monitoring and command listening. This is the correct way to disconnect. If you exit the program directly without calling stop, the robotic arm will enter a connection timeout error state.

**Examples:**

```python
api = HexDeviceApi(ws_url=args.url, control_hz=250)
arm = api.find_device_by_robot_type(16)
arm.stop()
```

### command_timeout_check

```python
def command_timeout_check(self, check_or_not: bool = True):
```

Sets whether to check for command timeout. When enabled, the arm will automatically brake if no commands are received within the timeout period (default 300ms).

**Examples:**

```python
# Enable command timeout checking (default)
arm.command_timeout_check(True)

# Disable command timeout checking
arm.command_timeout_check(False)
```

### motor_command

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

### clear_parking_stop

```python
def clear_parking_stop(self):
```

Clears the parking stop state.

**Examples:**

```python
arm.clear_parking_stop()
```

### enable_mit

```python
def enable_mit(self):
```

Enables MIT mode for torque control.

**Examples:**

```python
arm.enable_mit()

# Now MIT and TORQUE commands can be used
arm.motor_command(CommandType.TORQUE, [0.5, 0.3, 0.2, 0.1, 0.1, 0.0])
```

## Advanced Control Methods

> **Note:** This method is under development

<!--
### end_effector_control

```python
def end_effector_control(self, position: Union[List[float], Tuple[float, float, float]], orientation: Union[List[float], Tuple[float, float, float, float], None] = None, gravity_acc: Optional[List[float]] = None):
```

End effector control.

**Examples:**

```python
result = arm.end_effector_control(
    position=[0.5, 0.0, 0.0],  # Target position
    orientation=[0.0, 0.0, 0.0, 1.0],  # Target orientation as quaternion
    gravity_acc=[0.0, 0.0, 9.81]  # Gravity acceleration
)
```
-->

### joint_position_control

```python
def joint_position_control(self, joint_positions: List[float], velocities: List[float], accelerations: List[float]):
```

Joint position control with velocity and acceleration parameters.

**Parameters:**
- `joint_positions` (List[float]): Target joint positions in radians
- `velocities` (List[float]): Target joint velocities in radians per second
- `accelerations` (List[float]): Target joint accelerations in radians per second squared

**Examples:**

```python
arm.joint_position_control(
    joint_positions=[0.0, 0.5, 1.0, 0.0, 0.5, 0.0],
    velocities=[0.1, 0.1, 0.1, 0.0, 0.0, 0.0],
    accelerations=[0.5, 0.5, 0.5, 0.0, 0.0, 0.0]
)
```

### enable_free_drag

```python
def enable_free_drag(self, gravity_acc: list[float]):
```

Enables free drag mode.

**Examples:**

```python
arm.enable_free_drag(gravity_acc=[0.0, 0.0, 9.81])
```

### enable_zero_current_control

```python
def enable_zero_current_control(self):
```

Enables zero current control mode.

**Examples:**

```python
arm.enable_zero_current_control()
```

### compensated_mit_control

```python
def compensated_mit_control(self, mit_commands: List[MitMotorCommand], gravity_acc: List[float]):
```

Compensated MIT control with gravity compensation.

**Examples:**

```python
mit_commands = arm.construct_mit_command(
    pos=[0.0, 0.5, 1.0, 0.0, 0.5, 0.0],
    speed=[0.0, 0.0, 0.0, 0.0, 0.0, 0.0],
    torque=[0.0, 0.0, 0.0, 0.0, 0.0, 0.0],
    kp=[100.0] * 6,
    kd=[10.0] * 6
)
arm.compensated_mit_control(mit_commands, gravity_acc=[0.0, 0.0, 9.81])
```

## Configuration Methods

### get_arm_config

```python
def get_arm_config(self) -> Optional[ArmConfig]:
```

Gets the current robotic arm configuration.

**Examples:**

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

**Examples:**

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

**Examples:**

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

**Examples:**

```python
if not arm.check_motor_count_match():
    print("Warning: Motor count does not match configuration")
```

### get_arm_series

```python
def get_arm_series(self) -> int:
```

Gets the arm series ID.

**Examples:**

```python
series_id = arm.get_arm_series()
print(f"Arm series ID: {series_id}")
```

### get_arm_name

```python
def get_arm_name(self) -> Optional[str]:
```

Gets the arm name from configuration.

**Examples:**

```python
arm_name = arm.get_arm_name()
if arm_name:
    print(f"Arm name: {arm_name}")
```

### get_parking_stop_detail

```python
def get_parking_stop_detail(self) -> public_api_types_pb2.ParkingStopDetail:
```

Gets detailed parking stop information.

**Examples:**

```python
detail = arm.get_parking_stop_detail()
print(f"Parking stop: {detail}")
```

### get_session_holder

```python
def get_session_holder(self) -> int:
```

Gets the session holder ID.

**Examples:**

```python
session_id = arm.get_session_holder()
print(f"Session holder: {session_id}")
```

### get_my_session_id

```python
def get_my_session_id(self) -> int:
```

Gets this session's ID.

**Examples:**

```python
my_session = arm.get_my_session_id()
print(f"My session ID: {my_session}")
```

## Validation Methods

### validate_joint_positions

```python
def validate_joint_positions(self, positions: List[float], dt: float = 0.002) -> List[float]:
```

Validates and clamps joint positions to be within limits, considering velocity constraints.

**Examples:**

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

**Examples:**

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

Checks if command timeout has occurred.

**Examples:**

```python
if arm.is_timeout():
    print("Arm control has timed out")
else:
    print("Arm control is active")
```

## Motion History Management

<!--
### set_initial_positions

```python
def set_initial_positions(self, positions: List[float]):
```

Sets the initial positions for velocity limit calculations.

**Examples:**

```python
initial_pos = [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
arm.set_initial_positions(initial_pos)
```

### set_initial_velocities

```python
def set_initial_velocities(self, velocities: List[float]):
```

Sets the initial velocities for acceleration limit calculations.

**Examples:**

```python
initial_vel = [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
arm.set_initial_velocities(initial_vel)
```
-->

### get_last_positions

```python
def get_last_positions(self) -> Optional[List[float]]:
```

Gets the last positions of the arm joints.

**Examples:**

```python
positions = arm.get_last_positions()
if positions:
    print(f"Last positions: {positions}")
```

### get_last_velocities

```python
def get_last_velocities(self) -> Optional[List[float]]:
```

Gets the last velocities of the arm joints.

**Examples:**

```python
velocities = arm.get_last_velocities()
if velocities:
    print(f"Last velocities: {velocities}")
```

### clear_position_history

```python
def clear_position_history(self):
```

Clears the position history.

**Examples:**

```python
arm.clear_position_history()
```

### clear_velocity_history

```python
def clear_velocity_history(self):
```

Clears the velocity history.

**Examples:**

```python
arm.clear_velocity_history()
```

### clear_motion_history

```python
def clear_motion_history(self):
```

Clears all motion history.

**Examples:**

```python
arm.clear_motion_history()
```

## Inherited Methods

### From DeviceBase:

### From MotorBase:

## Usage Example

```python
import asyncio
from hex_device import HexDeviceApi

async def main():
    api = HexDeviceApi(ws_url="ws://192.168.1.1:8080", control_hz=250)
    await api.connect()

    arm = api.find_device_by_robot_type(16)
    if arm:
        arm.start()

        arm.motor_command(CommandType.POSITION, [0.0, 0.5, 1.0, 0.0, 0.5, 0.0])

        await asyncio.sleep(2)

        arm.stop()

    await api.disconnect()

if __name__ == "__main__":
    asyncio.run(main())
```

## Best Practices

1. Always call `start()` before sending commands
2. Set appropriate command timeout checking for safety
3. Validate joint positions before sending commands
4. Use MIT mode for precise torque control
5. Properly handle errors and timeouts

## Troubleshooting

Common issues and solutions:

1. **Arm not responding to commands**
   - Verify `start()` was called
   - Check if another controller has control
   - Verify network connection

2. **Joint position validation errors**
   - Ensure positions are within joint limits
   - Check velocity limits are not exceeded

3. **MIT mode not working**
   - Use `enable_mit()` to enable MIT mode first
   - Verify arm supports MIT commands

4. **Timeout errors**
   - Increase timeout period with `command_timeout_check()`
   - Reduce command interval
   - Check network latency
