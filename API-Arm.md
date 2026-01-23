The `Arm` class inherits from [DeviceBase](API-Common#DeviceBase) and [MotorBase](API-Motorbase), primarily implementing the control of robotic arm devices. This class corresponds to `ArmStatus` in the proto, managing arm status and motor control.

Supported robot types:
- `RtArmSaberD6x`: Saber 6-DOF robotic arm (ID: 14)
- `RtArmSaberD7x`: Saber 7-DOF robotic arm (ID: 15)
- `RtArmArcherD6Y_P1`: Archer 6-DOF robotic arm (ID: 16)
- `RtArmArcherY6L_V1`: Archer 6-DOF robotic arm (ID: 17)
- `RtArmArcherY6_H1`: Archer 6-DOF robotic arm (ID: 25)
- `RtArmFireflyY6_H1`: Firefly 6-DOF robotic arm (ID: 27)
- `RtHelloArcherY6_H1`: Hello Archer 6-DOF robotic arm (ID: 26)
- `RtHelloFireflyY6_H1`: Hello Firefly 6-DOF robotic arm (ID: 28)

# Arm
```python
class Arm(DeviceBase, MotorBase):
```

The common function can be found in: [DeviceBase](API-Common#DeviceBase) and [MotorBase](API-Motorbase).

## `__init__`
```python
def __init__(self, robot_type, motor_count, name: str = "Arm", control_hz: int = 500, send_message_callback=None):
```
Automatically called by HexDeviceApi to initialize the Arm robotic arm device.

**Parameters:**
- `robot_type`: Robot type (RobotType enum or int ID)
- `motor_count` (int): Number of motors
- `name` (str, optional): Device name, defaults to "Arm"
- `control_hz` (int, optional): Control frequency in Hz, defaults to 500
- `send_message_callback` (callable, optional): Callback function for sending messages

**Examples:**
```python
# Usually called internally by HexDeviceApi
arm = Arm(robot_type=16, motor_count=6, name="MyArm", control_hz=500)
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
    [0.0, 0.5, 1.0, 0.0, 0.5, 0.0],  # positions
    [0.0, 0.0, 0.0, 0.0, 0.0, 0.0],  # speeds
    [0.0, 0.0, 0.0, 0.0, 0.0, 0.0],  # torques
    [100.0] * 6,  # kp
    [10.0] * 6    # kd
)
arm.motor_command(CommandType.MIT, mit_commands)
```

## construct_mit_command
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

**Examples:**
```python
import numpy as np

# Using numpy arrays
mit_commands = arm.construct_mit_command(
    np.array([-0.3, -1.48, 2.86, 0.0, 0.0, 0.0]), 
    np.array([0.0, 0.0, 0.0, 0.0, 0.0, 0.0]), 
    np.array([0.0, 0.0, 0.0, 0.0, 0.0, 0.0]), 
    np.array([150.0, 150.0, 150.0, 150.0, 39.0, 39.0]), 
    np.array([12.0, 12.0, 12.0, 12.0, 0.8, 0.8])
)

# Using lists
mit_commands = arm.construct_mit_command(
    [0.3, -1.48, 2.86, 0.0, 0.0, 0.0], 
    [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], 
    [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], 
    [150.0, 150.0, 150.0, 150.0, 39.0, 39.0], 
    [12.0, 12.0, 12.0, 12.0, 0.8, 0.8]
)

# Use with motor_command
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
