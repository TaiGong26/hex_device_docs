The `Hands` class inherits from [OptionalDeviceBase](API-Common#optionaldevicebase) and [MotorBase](API-Common#motorbase), primarily implementing hand control and status management. This class processes the optional `hand_status` field from APIUp messages.

Supported hand types:
- `SdtHandGp100`: GP100 hand type
- `SdtHandGp80G1`: GP80G1 hand type
- `SdtHandGr100`: GR100 hand type

## `__init__`
```python
def __init__(self, device_id, device_type, motor_count, proto_version: tuple[int, int], send_message_callback, name: str = "Hands", control_hz: int = 250, read_only: bool = False):
```
Initializes a Hands device for robotic hand control.

**Parameters:**
- `device_id`: Device ID (from SecondaryDeviceStatus)
- `device_type`: Device type (SecondaryDeviceType enum, e.g., SdtHandGp100, SdtHandGp80G1, SdtHandGr100)
- `motor_count`: Number of motors in the hand
- `proto_version`: Protocol version as a tuple (major, minor)
- `send_message_callback`: Callback function for sending messages
- `name` (str, optional): Device name, defaults to "Hands"
- `control_hz` (int, optional): Control frequency in Hz, defaults to 250
- `read_only` (bool, optional): Whether this device is read-only, defaults to False

**Examples:**
```python
# Usually called internally by HexDeviceApi when creating hand devices
# Users typically don't need to call this directly
```

## command_timeout_check
```python
def command_timeout_check(self, check_or_not: bool = True):
```
Set whether to check command timeout.

**Parameters:**
- `check_or_not` (bool): Whether to enable command timeout checking

**Examples:**
```python
# Enable command timeout checking
hand.command_timeout_check(True)

# Disable command timeout checking
hand.command_timeout_check(False)
```

## motor_command
```python
def motor_command(self, command_type: CommandType, values: Union[List[bool], List[float], List[MitMotorCommand], np.ndarray]):
```
Set motor command for the hand. Supports position limiting and MIT command conversion.

**Parameters:**
- `command_type`: Command type (BRAKE, SPEED, POSITION, TORQUE, MIT)
- `values`: Command values (list or numpy array)

**Notes:**
- Position commands are automatically limited to hand joint limits
- MIT commands are automatically converted to position commands
- Values are validated for POSITION and SPEED command types

**Examples:**
```python
# Position command with automatic limiting
hand.motor_command(CommandType.POSITION, [0.0])

# Using numpy arrays
import numpy as np
positions = np.array([0.0])
hand.motor_command(CommandType.POSITION, positions)
```

## set_positon_step
```python
def set_positon_step(self, step: float):
```
Set position step for smooth position control, only affects the Position mode.

**Parameters:**
- `step` (float): Position step size (rad)

**Examples:**
```python
# Set smaller step for smoother control
hand.set_positon_step(0.01)

# Set larger step for faster movement
hand.set_positon_step(0.05)
```

## set_pos_torque
```python
def set_pos_torque(self, max_torque: float):
```
Set maximum torque for position control.

**Parameters:**
- `max_torque` (float): Maximum torque limit (Nm)

**Examples:**
```python
# Set conservative torque limit
hand.set_pos_torque(2.0)

# Set higher torque limit
hand.set_pos_torque(5.0)
```

## get_hand_type
```python
def get_hand_type(self) -> int:
```
Get the hand type.

**Returns:**
- `int`: Hand type enum value

**Examples:**
```python
hand_type = hand.get_hand_type()
print(f"Hand type: {hand_type}")
```

## get_joint_limits
```python
def get_joint_limits(self) -> List[float]:
```
Get the joint limits for the hand.

**Returns:**
- `List[float]`: List of joint limits [min, max, ...]

**Examples:**
```python
limits = hand.get_joint_limits()
print(f"Joint limits: {limits}")
```

## get_hands_summary
```python
def get_hands_summary(self) -> dict:
```
Get comprehensive hands device summary including motor data and configuration.

**Returns:**
- `dict`: Hands device summary containing:
  - Device information (name, has_new_data, last_update_time)
  - Hand configuration (hand_type, motor_count, control_hz)
  - Control settings (command_timeout_check, calibrated, api_control_initialized)
  - Motor data (positions, velocities, torques)

**Examples:**
```python
summary = hand.get_hands_summary()
print(f"Hand type: {summary['hand_type']}")
print(f"Motor count: {summary['motor_count']}")
print(f"Control frequency: {summary['control_hz']} Hz")
print(f"Current positions: {summary['motor_positions']}")
print(f"Current velocities: {summary['motor_velocities']}")
print(f"Current torques: {summary['motor_torques']}")
```

## Inherited Methods

The `Hands` class inherits all methods from `OptionalDeviceBase` and `MotorBase`, including:

### From OptionalDeviceBase:
- `has_new_data()` - Check for new data
- `get_device_summary()` - Get device status
- `supports_message_type()` - Check message type support

### From MotorBase:
- `get_motor_position(motor_index)` - Get individual motor position
- `get_motor_positions()` - Get all motor positions
- `get_motor_velocity(motor_index)` - Get individual motor velocity
- `get_motor_velocities()` - Get all motor velocities
- `get_motor_torque(motor_index)` - Get individual motor torque
- `get_motor_torques()` - Get all motor torques
- `get_motor_state(motor_index)` - Get motor state
- `get_motor_summary()` - Get motor group summary
- `construct_mit_command()` - Constructs MIT command from numpy array or list

**Examples:**
```python
# Check for new data
if hand.has_new_data():
    # Get current hand state
    positions = hand.get_motor_positions()
    velocities = hand.get_motor_velocities()
    torques = hand.get_motor_torques()
    
    # Process hand data
    process_hand_data(positions, velocities, torques)
    
# Check individual motor status
for i in range(hand.motor_count):
    state = hand.get_motor_state(i)
    if state == "error":
        print(f"Motor {i} has error")
```
