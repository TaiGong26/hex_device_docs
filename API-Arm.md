The `Arm` class inherits from [DeviceBase](Function-common#devicebase) and [MotorBase](Function-common#motorbase), primarily implementing the control of robotic arm devices. This class corresponds to `ArmStatus` in the proto, managing arm status and motor control.

Supported robot types:
- `RtArmArcherD6Y`: Archer 6-DOF robotic arm (ID: 16)
- `RtArmSaberD6X`: Saber 6-DOF robotic arm (ID: 14)  

# Arm
```python
class Arm(DeviceBase, MotorBase):
```

## `__init__`
```python
def __init__(self, robot_type, motor_count, name: str = "ArmArcher", control_hz: int = 500, send_message_callback=None):
```
Automatically called by HexDeviceApi to initialize the Arm robotic arm device.

Examples:
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
def motor_command(self, command_type: CommandType, values: Union[List[bool], List[float], List[MitMotorCommand]]):
```
Sets robotic arm motor commands with automatic validation for position and velocity commands.  
> **Warning!!!!**Only once of command can be set at the same time.

Examples:
```python
# Set joint position commands (with automatic validation)
arm.motor_command(CommandType.POSITION, [0.0, 0.5, 1.0, 0.0, 0.5, 0.0])

# Set joint velocity commands (with automatic validation)
arm.motor_command(CommandType.SPEED, [0.1, -0.1, 0.2, 0.0, -0.1, 0.0])

# Set brake commands
arm.motor_command(CommandType.BRAKE, [True] * 6)

# Set torque commands
arm.motor_command(CommandType.TORQUE, [0.5, 0.3, 0.2, 0.1, 0.1, 0.0])
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

Examples:
```python
id = arm.get_arm_config()
if id == 0:
    print(f"No one is controlling, you can use start() to try to get control")
```

## get_my_session_id
```python
def get_my_session_id(self) -> int:
```
Gets the session ID of the current connection.

Examples:
```python
id = arm.get_my_session_id()
print(f"My session id is {id}")
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
Gets joint position limits [min, max] for each joint.

Examples:
```python
limits = arm.get_joint_limits()
if limits:
    for i, (min_pos, max_pos) in enumerate(limits):
        print(f"Joint {i}: [{min_pos:.3f}, {max_pos:.3f}] rad")
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

Examples:
```python
arm.clear_motion_history()
print("All motion history cleared")
```
