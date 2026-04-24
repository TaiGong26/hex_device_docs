# MotorBase
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
4. [Command Methods](#command-methods)
    - [motor_command](#motor_command)
    - [mit_motor_command](#mit_motor_command)
    - [construct_mit_command](#construct_mit_command)
    - [construct_speedWithMaxCurrent_command](#construct_speedWithMaxCurrent_command)
    <!-- - [construct_posVelAcc_command](#construct_posVelAcc_command) -->
5. [Conversion Methods](#conversion-methods)
   - [convert_positions_to_rad](#convert_positions_to_rad)
   - [convert_rad_to_positions](#convert_rad_to_positions)
6. [Data Checking](#data-checking)
   - [has_new_data](#has_new_data)
7. [State Methods](#state-methods)
   - [get_motor_state](#get_motor_state)
   - [get_motor_states](#get_motor_states)
8. [Status Methods](#status-methods)
   - [get_simple_motor_status](#get_simple_motor_status)
   - [get_motor_status](#get_motor_status)
9. [Position Methods](#position-methods)
    - [get_motor_position](#get_motor_position)
    - [get_motor_positions](#get_motor_positions)
    - [get_motor_encoder_positions](#get_motor_encoder_positions)
    - [get_encoders_to_zero](#get_encoders_to_zero)
10. [Velocity Methods](#velocity-methods)
    - [get_motor_velocity](#get_motor_velocity)
    - [get_motor_velocities](#get_motor_velocities)
11. [Torque Methods](#torque-methods)
    - [get_motor_torque](#get_motor_torque)
    - [get_motor_torques](#get_motor_torques)
12. [Error Methods](#error-methods)
   - [get_motor_error_codes](#get_motor_error_codes)
13. [Temperature Methods](#temperature-methods)
    - [get_motor_driver_temperatures](#get_motor_driver_temperatures)
    - [get_motor_driver_temperature](#get_motor_driver_temperature)
    - [get_motor_temperatures](#get_motor_temperatures)
    - [get_motor_temperature](#get_motor_temperature)
    - [get_motor_warnings](#get_motor_warnings)
14. [Voltage Methods](#voltage-methods)
    - [get_motor_voltage](#get_motor_voltage)
    - [get_motor_voltages](#get_motor_voltages)
15. [Motor Parameters](#motor-parameters)
    - [get_motor_pulse_per_rotation](#get_motor_pulse_per_rotation)
    - [get_motor_pulse_per_rotations](#get_motor_pulse_per_rotations)
    - [get_motor_wheel_radius](#get_motor_wheel_radius)
    - [get_motor_wheel_radii](#get_motor_wheel_radii)
16. [Properties](#properties)
   - [cache_motion_data](#cache_motion_data)
   - [cache_positions](#cache_positions)
   - [cache_velocities](#cache_velocities)
   - [cache_torques](#cache_torques)
   - [target_positions](#target_positions)
   - [target_velocities](#target_velocities)
   - [target_torques](#target_torques)
17. [Summary Methods](#summary-methods)
    - [get_motor_summary](#get_motor_summary)
18. [Utility Methods](#utility-methods)
    - [flush_motor_data](#flush_motor_data)

## Overview

MotorBase is the base class for devices with motors, providing common motor control functionality for devices such as chassis and robotic arms.


## Class Definition
```python
class MotorBase:
```

## `__init__`
```python
def __init__(self, motor_count: int, proto_version: tuple[int, int], name: str = "", convert_positions_to_rad_func: Optional[Callable[[np.ndarray, np.ndarray], np.ndarray]] = None, convert_rad_to_positions_func: Optional[Callable[[np.ndarray, np.ndarray], np.ndarray]] = None):
```
Automatically called by HexDeviceApi, passing in the motor count and motor group name.

**Parameters:**
- `motor_count` (int): Number of motors
- `proto_version` (tuple[int, int]): Protocol version (major, minor)
- `name` (str, optional): Motor group name. Defaults to empty string
- `convert_positions_to_rad_func` (Optional[Callable]): Function to convert positions to radians
- `convert_rad_to_positions_func` (Optional[Callable]): Function to convert radians to positions


## Command Methods

> **Note:** The actual method name in code is `motor_command`, but when using devices that inherit from MotorBase (like Arm, Chassis, etc.), you should call it via the device instance: `device.motor_command()`. Since MotorBase is an abstract base class for devices with motors, the command is sent to the actual device, not to a "motor" object directly.

### motor_command
```python
def motor_command(self, command_type: CommandType, values: Union[List[bool], List[float], List[MitMotorCommand], List[SpeedWithMaxCurrentMotorCommand], List[PosVelAccCommand], np.ndarray]):
```
Sets motor commands for the device, supporting seven command types: BRAKE, SPEED, POSITION, TORQUE, MIT, SPEED_WITH_MAX_CURRENT.

**Parameters:**
- `command_type` (CommandType): Type of command:
  - `BRAKE`: Brake control (values determines motor count only)
  - `SPEED`: Speed control (rad/s)
  - `POSITION`: Position control (rad)
  - `TORQUE`: Torque control (Nm)
  - `MIT`: MIT control with PID (List[MitMotorCommand])
  - `SPEED_WITH_MAX_CURRENT`: Speed control with max current (List[SpeedWithMaxCurrentMotorCommand])
  <!-- - `POS_VEL_ACC`: Position-velocity-acceleration control (List[PosVelAccCommand]) -->
- `values`: Command values:
  - BRAKE: `List[bool]` - brake states
  - SPEED: `List[float]` - target speeds (rad/s)
  - POSITION: `List[float]` - target positions (rad)
  - TORQUE: `List[float]` - target torques (Nm)
  - MIT: `List[MitMotorCommand]` - MIT commands with position, speed, torque, kp, kd
  - SPEED_WITH_MAX_CURRENT: `List[SpeedWithMaxCurrentMotorCommand]` - speed commands with max current
  <!-- - POS_VEL_ACC: `List[PosVelAccCommand]` - position-velocity-acceleration commands -->

**Motor Command Support by Device:**

| Command | Arm | Chassis | ZetaLift | LinearLift | Hands |
|---------|-----|---------|----------|------------|-------|
| BRAKE | ✓ | ✓ | ✓ | ✓ | ✓ |
| SPEED | ✓ | ✓ | ✓ | ✗ | ✓ |
| POSITION | ✓ | ✗ | ✓ | ✓ | ✓ |
| TORQUE | ✓* | ✗ | ✗ | ✗ | ✓ |
| MIT | ✓* | ✗ | ✗ | ✗ | ✓ |
| SPEED_WITH_MAX_CURRENT | ✗ | ✓ | ✗ | ✗ | ✗ |
<!-- | POS_VEL_ACC | ✗ | ✗ | ✗ | ✗ | ✗ | -->

*Note: Arm's TORQUE and MIT commands require `enable_zero_current_control()` to be called first.*

Examples:
```python
from hex_device.motor_base import CommandType, MitMotorCommand

# BRAKE command - values only determine motor count
device.motor_command(CommandType.BRAKE, [True] * motor_count)

# SPEED command - control motor speeds (rad/s)
device.motor_command(CommandType.SPEED, [1.0, -1.0, 0.5])

# POSITION command - control motor positions (rad)
device.motor_command(CommandType.POSITION, [0.0, 1.57, 3.14])

# TORQUE command - control motor torques (Nm)
device.motor_command(CommandType.TORQUE, [0.5, 0.3, 0.0])

# set mit command
mit_commands = device.construct_mit_command(
    np.array([-0.3, -1.48, 2.86, 0.0, 0.0, 0.0]),
    np.array([0.0, 0.0, 0.0, 0.0, 0.0, 0.0]),
    np.array([0.0, 0.0, 0.0, 0.0, 0.0, 0.0]),
    np.array([150.0, 150.0, 150.0, 150.0, 39.0, 39.0]),
    np.array([12.0, 12.0, 12.0, 12.0, 0.8, 0.8])
)

device.motor_command(CommandType.MIT, mit_commands)

# SPEED_WITH_MAX_CURRENT command - speed control with max current limit (A)
speed_commands = device.construct_speedWithMaxCurrent_command(
    speed=np.array([1.0, 2.0, 3.0]),
    max_current=np.array([10.0, 10.0, 10.0])
)

device.motor_command(CommandType.SPEED_WITH_MAX_CURRENT, speed_commands)

```

### mit_motor_command
```python
def mit_motor_command(self, mit_commands: List[MitMotorCommand]):
```
Convenience method for MIT motor commands. Internally calls `motor_command(CommandType.MIT, mit_commands)`.

**Parameters:**
- `mit_commands` (List[MitMotorCommand]): List of MIT motor commands, each containing:
  - `position`: Target position (rad)
  - `speed`: Target speed (rad/s)
  - `torque`: Target torque (Nm)
  - `kp`: Proportional gain
  - `kd`: Derivative gain

Examples:
```python
from hex_device.motor_base import MitMotorCommand

mit_cmds = [
    MitMotorCommand(torque=0.5, speed=1.0, position=0.0, kp=10.0, kd=1.0),
    MitMotorCommand(torque=0.3, speed=0.5, position=1.57, kp=8.0, kd=0.8)
]
device.mit_motor_command(mit_cmds)
```

### construct_mit_command
```python
def construct_mit_command(self,
            pos: Union[np.ndarray, List[float]],
            speed: Union[np.ndarray, List[float]],
            torque: Union[np.ndarray, List[float]],
            kp: Union[np.ndarray, List[float]],
            kd: Union[np.ndarray, List[float]]
        ) -> List[MitMotorCommand]:
```
Constructs MIT command from numpy array or list. MIT commands allow simultaneous control of position, speed, torque with PID gains.

**Parameters:**
- `pos` (Union[np.ndarray, List[float]]): Target positions for each joint (rad)
- `speed` (Union[np.ndarray, List[float]]): Target speeds for each joint (rad/s)
- `torque` (Union[np.ndarray, List[float]]): Target torques for each joint (Nm)
- `kp` (Union[np.ndarray, List[float]]): Proportional gains for each joint
- `kd` (Union[np.ndarray, List[float]]): Derivative gains for each joint

**Returns:**
- `List[MitMotorCommand]`: List of MIT motor commands

Examples:
```python
from hex_device.motor_base import CommandType
import numpy as np

# Using numpy arrays to construct MIT commands
mit_commands = device.construct_mit_command(
    np.array([-0.3, -1.48, 2.86, 0.0, 0.0, 0.0]),
    np.array([0.0, 0.0, 0.0, 0.0, 0.0, 0.0]),
    np.array([0.0, 0.0, 0.0, 0.0, 0.0, 0.0]),
    np.array([150.0, 150.0, 150.0, 150.0, 39.0, 39.0]),
    np.array([12.0, 12.0, 12.0, 12.0, 0.8, 0.8])
)

# Use with motor_command
device.motor_command(CommandType.MIT, mit_commands)
```

### construct_speedWithMaxCurrent_command
```python
def construct_speedWithMaxCurrent_command(self,
            speed: Union[np.ndarray, List[float]],
            max_current: Union[np.ndarray, List[float]]
        ) -> List[SpeedWithMaxCurrentMotorCommand]:
```
Constructs speed with max current command for each motor.

**Parameters:**
- `speed` (Union[np.ndarray, List[float]]): Target speeds for each joint (rad/s)
- `max_current` (Union[np.ndarray, List[float]]): Maximum current for each joint (A)

**Returns:**
- `List[SpeedWithMaxCurrentMotorCommand]`: List of speed with max current commands

**Examples:**
```python
from hex_device.motor_base import CommandType
import numpy as np

# Construct speed commands with max current
speed_commands = device.construct_speedWithMaxCurrent_command(
    speed=np.array([1.0, 2.0, 3.0]),
    max_current=np.array([10.0, 10.0, 10.0])
)

device.motor_command(CommandType.SPEED_WITH_MAX_CURRENT, speed_commands)
```
<!-- 
### construct_posVelAcc_command
```python
def construct_posVelAcc_command(self,
            position: Union[np.ndarray, List[float]],
            velocity: Union[np.ndarray, List[float]],
            acceleration: Union[np.ndarray, List[float]]
        ) -> List[PosVelAccCommand]:
```
Constructs position-velocity-acceleration command for each motor.

**Parameters:**
- `position` (Union[np.ndarray, List[float]]): Target positions for each joint (rad)
- `velocity` (Union[np.ndarray, List[float]]): Target velocities for each joint (rad/s)
- `acceleration` (Union[np.ndarray, List[float]]): Target accelerations for each joint (rad/s²)

**Returns:**
- `List[PosVelAccCommand]`: List of position-velocity-acceleration commands

**Examples:**
```python
from hex_device.motor_base import CommandType
import numpy as np

# Construct position-velocity-acceleration commands
pos_vel_acc_commands = device.construct_posVelAcc_command(
    position=np.array([0.0, 1.57, 3.14]),
    velocity=np.array([0.5, 0.5, 0.5]),
    acceleration=np.array([1.0, 1.0, 1.0])
)

device.motor_command(CommandType.POS_VEL_ACC, pos_vel_acc_commands)
``` 
-->


## Conversion Methods

### convert_positions_to_rad
```python
def convert_positions_to_rad(self, positions: np.ndarray, pulse_per_rotation: np.ndarray) -> np.ndarray:
```
Converts encoder positions to radians. This method provides a default implementation but can be overridden by providing a custom function during instance initialization.

**Parameters:**
- `positions` (np.ndarray): Position values in encoder pulses
- `pulse_per_rotation` (np.ndarray): Pulse per rotation values

**Returns:**
- `np.ndarray`: Position values in radians

**Examples:**
```python
positions_rad = motor.convert_positions_to_rad(positions, pulse_per_rotation)
print(f"Positions in radians: {positions_rad}")
```

### convert_rad_to_positions
```python
def convert_rad_to_positions(self, positions: np.ndarray, pulse_per_rotation: np.ndarray) -> np.ndarray:
```
Converts radian positions to encoder positions. This method provides a default implementation but can be overridden by providing a custom function during instance initialization.

**Parameters:**
- `positions` (np.ndarray): Position values in radians
- `pulse_per_rotation` (np.ndarray): Pulse per rotation values

**Returns:**
- `np.ndarray`: Position values in encoder pulses

**Examples:**
```python
positions = motor.convert_rad_to_positions(positions_rad, pulse_per_rotation)
print(f"Positions in encoder pulses: {positions}")
```


## Data Checking

### has_new_data
```python
def has_new_data(self) -> bool:
```
Checks if data queue is empty.

Examples:
```python
for device in api.device_list:
    if device.has_new_data():
        # do something for device...
        pass
```


## State Methods

### get_motor_state
```python
def get_motor_state(self, motor_index: int) -> Optional[str]:
```
Gets the status of the specified motor, returning "normal" or "error". Always returns the latest frame of data.

Examples:
```python
state = motor.get_motor_state(0)
if state == "error":
    print(f"Motor 0 is in error state")
```

### get_motor_states
```python
def get_motor_states(self) -> Optional[List[str]]:
```
Gets the status list of all motors, returning "normal" or "error" for each motor. Always returns the latest frame of data.

Examples:
```python
states = motor.get_motor_states()
if states is not None:
    for i, state in enumerate(states):
        if state == "error":
            print(f"Motor {i} is in error state")
```


## Status Methods

### get_simple_motor_status
```python
def get_simple_motor_status(self, pop: bool = True) -> Optional[Dict[str, Any]]:
```
Gets simple motor status dictionary or None if queue is empty. If not pop, always returns the latest frame of data. The dictionary contains 'pos' (positions), 'vel' (velocities), 'eff' (torques), and 'ts' (timestamp).

**Parameters:**
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

Examples:
```python
status = device.get_simple_motor_status()
if status is not None:
    print(f"Simple motor status: {status}")
```

### get_motor_status
```python
def get_motor_status(self, motor_index: int, pop: bool = True) -> Optional[Dict[str, Any]]:
```
Gets detailed status information for the specified motor, including current status and target commands. If not pop, always returns the latest frame of data. Returns None if queue is empty.

**Parameters:**
- `motor_index` (int): Index of the motor
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

Examples:
```python
status = motor.get_motor_status(0)
if status is not None:
    print(f"Motor 0 state: {status['state']}")
    print(f"Position: {status['position']} rad")
    print(f"Velocity: {status['velocity']} rad/s")
    print(f"Target velocity: {status['target_velocity']} rad/s")
    if status['error_code'] is not None:
        print(f"Error code: {status['error_code']}")
```


## Position Methods

### get_motor_position
```python
def get_motor_position(self, motor_index: int, pop: bool = True) -> Optional[float]:
```
Gets the position of the specified motor (unit: rad) or None if queue is empty. If not pop, always returns the latest frame of data.

**Parameters:**
- `motor_index` (int): Index of the motor
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

Examples:
```python
position = motor.get_motor_position(0)
if position is not None:
    print(f"Motor 0 position: {position} rad")
```

### get_motor_positions
```python
def get_motor_positions(self, pop: bool = True) -> Optional[List[float]]:
```
Gets the position list of all motors (unit: rad) or None if queue is empty. If not pop, always returns the latest frame of data.

**Parameters:**
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

Examples:
```python
positions = motor.get_motor_positions()
if positions is not None:
    for i, pos in enumerate(positions):
        print(f"Motor {i}: {pos} rad")
```

### get_motor_encoder_positions
```python
def get_motor_encoder_positions(self, pop: bool = True) -> Optional[np.ndarray]:
```
Gets the encoder position list of all motors (unit: pulse) or None if queue is empty. If not pop, always returns the latest frame of data.

**Parameters:**
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

Examples:
```python
positions = motor.get_motor_encoder_positions()
if positions is not None:
    for i, pos in enumerate(positions):
        print(f"Motor {i}: {pos} pulse")
```

### get_encoders_to_zero
```python
def get_encoders_to_zero(self, pop: bool = True) -> Optional[List[float]]:
```
Retrieves the encoder values from the current position to the zero point, which can be used to check the difference between the current position and the software's zero position. Note that this value is only meaningful for the robotic arm. If not pop, always returns the latest frame of data.

**Parameters:**
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

Examples:
```python
encoders_bias = device.get_encoders_to_zero()
if encoders_bias is not None:
    print(f"Encoders to zero: {encoders_bias}")
```


## Velocity Methods

### get_motor_velocity
```python
def get_motor_velocity(self, motor_index: int, pop: bool = True) -> Optional[float]:
```
Gets the velocity of the specified motor (unit: rad/s). If not pop, always returns the latest frame of data.

**Parameters:**
- `motor_index` (int): Index of the motor
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

Examples:
```python
velocity = motor.get_motor_velocity(0)
if velocity is not None:
    print(f"Motor 0 velocity: {velocity} rad/s")
```

### get_motor_velocities
```python
def get_motor_velocities(self, pop: bool = True) -> Optional[List[float]]:
```
Gets the velocity list of all motors (unit: rad/s). If not pop, always returns the latest frame of data.

**Parameters:**
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

Examples:
```python
velocities = motor.get_motor_velocities()
if velocities is not None:
    avg_velocity = sum(velocities) / len(velocities)
    print(f"Average velocity: {avg_velocity} rad/s")
```


## Torque Methods

### get_motor_torque
```python
def get_motor_torque(self, motor_index: int, pop: bool = True) -> Optional[float]:
```
Gets the torque of the specified motor (unit: Nm). If not pop, always returns the latest frame of data.

**Parameters:**
- `motor_index` (int): Index of the motor
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

Examples:
```python
torque = motor.get_motor_torque(0)
if torque is not None:
    print(f"Motor 0 torque: {torque} Nm")
```

### get_motor_torques
```python
def get_motor_torques(self, pop: bool = True) -> Optional[List[float]]:
```
Gets the torque list of all motors (unit: Nm). If not pop, always returns the latest frame of data.

**Parameters:**
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

Examples:
```python
torques = motor.get_motor_torques()
if torques is not None:
    total_torque = sum(torques)
    print(f"Total torque: {total_torque} Nm")
```


## Error Methods

### get_motor_error_codes
```python
def get_motor_error_codes(self) -> Optional[List[Optional[int]]]:
```
Gets all motor error codes. Returns a list of error codes (None for motors without errors) or None if data is not available. Always returns the latest frame of data.

Examples:
```python
error_codes = device.get_motor_error_codes()
if error_codes is not None:
    for i, code in enumerate(error_codes):
        if code is not None:
            print(f"Motor {i} error code: {code}")
```


## Temperature Methods

### get_motor_driver_temperatures
```python
def get_motor_driver_temperatures(self) -> Optional[np.ndarray]:
```
Gets the driver temperature list of all motors (unit: degC). Always returns the latest frame of data.

Examples:
```python
temps = motor.get_motor_driver_temperatures()
if temps is not None:
    for i, temp in enumerate(temps):
        if temp > 80:
            print(f"Warning: Driver {i} overheating at {temp}degC")
```

### get_motor_driver_temperature
```python
def get_motor_driver_temperature(self, motor_index: int) -> Optional[float]:
```
Gets the driver temperature of the specified motor (unit: degC). Always returns the latest frame of data.

**Parameters:**
- `motor_index` (int): Index of the motor

Examples:
```python
temp = motor.get_motor_driver_temperature(0)
if temp is not None and temp > 80:
    print(f"Warning: Driver 0 overheating at {temp}degC")
```

### get_motor_temperatures
```python
def get_motor_temperatures(self) -> Optional[np.ndarray]:
```
Gets the temperature list of all motors (unit: degC). Always returns the latest frame of data.

Examples:
```python
temps = motor.get_motor_temperatures()
if temps is not None:
    for i, temp in enumerate(temps):
        if temp > 70:
            print(f"Warning: Motor {i} overheating at {temp}degC")
```

### get_motor_temperature
```python
def get_motor_temperature(self, motor_index: int) -> Optional[float]:
```
Gets the temperature of the specified motor (unit: degC). Always returns the latest frame of data.

**Parameters:**
- `motor_index` (int): Index of the motor

Examples:
```python
temp = motor.get_motor_temperature(0)
if temp is not None and temp > 70:
    print(f"Warning: Motor 0 overheating at {temp}degC")
```

### get_motor_warnings
```python
def get_motor_warnings(self) -> Optional[List[str]]:
```
Gets all motor warnings.

**Returns:**
- `Optional[List[str]]`: List of motor warnings, or None if no warnings

Examples:
```python
warnings = motor.get_motor_warnings()
if warnings is not None:
    for warning in warnings:
        print(f"Warning: {warning}")
```


## Voltage Methods

### get_motor_voltage
```python
def get_motor_voltage(self, motor_index: int) -> Optional[float]:
```
Gets the voltage of the specified motor (unit: V). Always returns the latest frame of data.

**Parameters:**
- `motor_index` (int): Index of the motor

Examples:
```python
voltage = motor.get_motor_voltage(0)
if voltage is not None:
    print(f"Motor 0 voltage: {voltage} V")
```

### get_motor_voltages
```python
def get_motor_voltages(self) -> Optional[np.ndarray]:
```
Gets the voltage list of all motors (unit: V). Always returns the latest frame of data.

Examples:
```python
voltages = motor.get_motor_voltages()
if voltages is not None:
    for i, voltage in enumerate(voltages):
        print(f"Motor {i} voltage: {voltage} V")
```


## Motor Parameters

### get_motor_pulse_per_rotation
```python
def get_motor_pulse_per_rotation(self, motor_index: int) -> Optional[float]:
```
Gets the pulse per rotation of the specified motor. Returns None if not set.

**Parameters:**
- `motor_index` (int): Index of the motor

Examples:
```python
ppr = motor.get_motor_pulse_per_rotation(0)
if ppr is not None:
    print(f"Motor 0 pulses per rotation: {ppr}")
```

### get_motor_pulse_per_rotations
```python
def get_motor_pulse_per_rotations(self) -> Optional[np.ndarray]:
```
Gets the pulse per rotation list of all motors. Returns None if not set.

Examples:
```python
pprs = motor.get_motor_pulse_per_rotations()
if pprs is not None:
    for i, ppr in enumerate(pprs):
        print(f"Motor {i} pulses per rotation: {ppr}")
```

### get_motor_wheel_radius
```python
def get_motor_wheel_radius(self, motor_index: int) -> Optional[float]:
```
Gets the wheel radius of the specified motor (unit: m). Returns None if not set.

**Parameters:**
- `motor_index` (int): Index of the motor

Examples:
```python
radius = motor.get_motor_wheel_radius(0)
if radius is not None:
    linear_velocity = motor.get_motor_velocity(0) * radius
```

### get_motor_wheel_radii
```python
def get_motor_wheel_radii(self) -> Optional[np.ndarray]:
```
Gets the wheel radius list of all motors (unit: m). Returns None if not set.

Examples:
```python
radii = motor.get_motor_wheel_radii()
if radii is not None:
    for i, radius in enumerate(radii):
        print(f"Motor {i} wheel radius: {radius} m")
```


## Properties

### cache_motion_data
```python
@property
def cache_motion_data(self) -> Tuple[Optional[np.ndarray], Optional[np.ndarray], Optional[np.ndarray]]:
```
Get all motor cache motion data (positions radians, velocities rad/s, torques Nm).

**Returns:**
- `Tuple[Optional[np.ndarray], Optional[np.ndarray], Optional[np.ndarray]]`: A tuple of (positions, velocities, torques)

**Examples:**
```python
positions, velocities, torques = motor.cache_motion_data
if positions is not None:
    print(f"Cached positions: {positions}")
if velocities is not None:
    print(f"Cached velocities: {velocities}")
if torques is not None:
    print(f"Cached torques: {torques}")
```

### cache_positions
```python
@property
def cache_positions(self) -> Optional[np.ndarray]:
```
Get all motor cache positions (rad).

**Returns:**
- `Optional[np.ndarray]`: Array of cached positions or None

**Examples:**
```python
positions = motor.cache_positions
if positions is not None:
    print(f"Cached positions: {positions}")
```

### cache_velocities
```python
@property
def cache_velocities(self) -> Optional[np.ndarray]:
```
Get all motor cache velocities (rad/s).

**Returns:**
- `Optional[np.ndarray]`: Array of cached velocities or None

**Examples:**
```python
velocities = motor.cache_velocities
if velocities is not None:
    print(f"Cached velocities: {velocities}")
```

### cache_torques
```python
@property
def cache_torques(self) -> Optional[np.ndarray]:
```
Get all motor cache torques (Nm).

**Returns:**
- `Optional[np.ndarray]`: Array of cached torques or None

**Examples:**
```python
torques = motor.cache_torques
if torques is not None:
    print(f"Cached torques: {torques}")
```

### target_positions
```python
@property
def target_positions(self) -> np.ndarray:
```
Retrieve all the commands currently sent by the motors (rad).

Examples:
```python
print(device.target_positions)
```

### target_velocities
```python
@property
def target_velocities(self) -> np.ndarray:
```
Retrieve all the commands currently sent by the motors (rad/s).

Examples:
```python
target_velocities = motor.target_velocities
print(f"All motor target_velocities: {target_velocities}")
```

### target_torques
```python
@property
def target_torques(self) -> np.ndarray:
```
Retrieve all the commands currently sent by the motors (Nm).

Examples:
```python
target_torques = motor.target_torques
print(f"All motor target_torques: {target_torques}")
```


## Summary Methods

### get_motor_summary
```python
def get_motor_summary(self) -> Optional[Dict[str, Any]]:
```
Gets the motor group status summary, containing status information for all motors. Returns None if no data available.

**Returns:**
- `Optional[Dict[str, Any]]`: Dictionary containing:
  - `name` (str): Device name
  - `motor_count` (int): Number of motors
  - `positions` (List[float]): Motor positions (rad)
  - `velocities` (List[float]): Motor velocities (rad/s)
  - `torques` (List[float]): Motor torques (Nm)
  - `error_codes` (List[Optional[int]]): Motor error codes (some elements may be None)
  - `driver_temperature` (List[float]): Driver temperatures (degC)
  - `motor_temperature` (List[float]): Motor temperatures (degC)
  - `voltage` (List[float]): Motor voltages (V)
  - `pulse_per_rotation` (Optional[List[float]]): Pulses per rotation
  - `wheel_radius` (Optional[List[float]]): Wheel radii (m)
  - `last_update_time` (Optional[Dict]): Last update timestamp

Examples:
```python
summary = motor.get_motor_summary()
if summary is not None:
    print(f"Motor count: {summary['motor_count']}")
    print(f"Positions: {summary['positions']}")
    print(f"Velocities: {summary['velocities']}")
    print(f"Torques: {summary['torques']}")
```


## Utility Methods

### flush_motor_data
```python
def flush_motor_data(self):
```
Clears all motor data queues in MotorBase. This method removes all data from all queues.

Examples:
```python
motor.flush_motor_data()
```
<!-- 
## Best Practices

1. **Use appropriate data retrieval methods**
   - Use `pop=True` when processing data in order
   - Use `pop=False` when you only need the latest value

2. **Monitor motor health regularly**
   - Check temperatures and error codes periodically

3. **Use proper command types**
   - Choose the appropriate command type for your use case

4. **Handle None returns properly**
   - Always check if methods return None before processing data

5. **Use numpy arrays for large data sets**
   - Numpy arrays are more efficient for large motor counts

## Troubleshooting

1. **Data Queue Empty**
   - **Symptom**: Methods return None
   - **Solution**: Check connection and ensure data is being received

2. **Motor Overheating**
   - **Symptom**: High temperature readings
   - **Solution**: Reduce load, increase cooling, or implement rest periods

3. **Motor Error States**
   - **Symptom**: Error codes returned
   - **Solution**: Check error codes and refer to motor documentation

4. **Command Not Executing**
   - **Symptom**: Motor not responding to commands
   - **Cause**: Incorrect command type or values out of range
   - **Solution**: Verify command type and parameter ranges
 -->
