The `Gamepad` class inherits from [OptionalDeviceBase](API-Common#OptionalDeviceBase), primarily implementing gamepad data reading and status management. This class processes the optional `gamepad_read` field from APIUp messages.

Supported device types:
- `SdtGamepad`: Gamepad device type

# Gamepad
```python
class Gamepad(OptionalDeviceBase):
```

The common function can be found in: [OptionalDeviceBase](API-Common#OptionalDeviceBase).

## `__init__`
```python
def __init__(self, device_id, device_type, send_message_callback, name: str = "Gamepad", control_hz: int = 250, read_only: bool = True):
```
Automatically called by HexDeviceApi to initialize the Gamepad device.

**Parameters:**
- `device_id`: Device ID from SecondaryDeviceStatus
- `device_type`: Device type (SecondaryDeviceType enum, e.g., SdtGamepad)
- `send_message_callback`: Callback function for sending messages
- `name` (str, optional): Device name, defaults to "Gamepad"
- `control_hz` (int, optional): Control frequency in Hz, defaults to 250
- `read_only` (bool, optional): Whether this device is read-only, defaults to True

Examples:
```python
# Usually called internally by HexDeviceApi when creating Gamepad devices
# Users typically don't need to call this directly
```

## get_gamepad_read
```python
def get_gamepad_read(self) -> Optional[public_api_types_pb2.GamepadRead]:
```
Gets the current gamepad read data.

**Returns:**
- `Optional[GamepadRead]`: GamepadRead protobuf message or None if no data available. The GamepadRead message contains:
  - `left_stick_x` (float): Left stick X axis value (-1.0 to 1.0)
  - `left_stick_y` (float): Left stick Y axis value (-1.0 to 1.0)
  - `right_stick_x` (float): Right stick X axis value (-1.0 to 1.0)
  - `right_stick_y` (float): Right stick Y axis value (-1.0 to 1.0)
  - `left_bumper` (bool): Left bumper button state
  - `right_bumper` (bool): Right bumper button state
  - `left_trigger` (float): Left trigger value (0.0 to 1.0)
  - `right_trigger` (float): Right trigger value (0.0 to 1.0)
  - `a_button` (bool): A button state
  - `b_button` (bool): B button state
  - `x_button` (bool): X button state
  - `y_button` (bool): Y button state
  - `select_button` (bool): Select button state
  - `start_button` (bool): Start button state
  - `left_stick_button` (bool): Left stick button state
  - `right_stick_button` (bool): Right stick button state
  - `dpad_up` (bool): D-pad up button state
  - `dpad_down` (bool): D-pad down button state
  - `dpad_left` (bool): D-pad left button state
  - `dpad_right` (bool): D-pad right button state

Examples:
```python
gamepad_read = gamepad.get_gamepad_read()
if gamepad_read is not None:
    print(f"Left stick: ({gamepad_read.left_stick_x}, {gamepad_read.left_stick_y})")
    print(f"Right stick: ({gamepad_read.right_stick_x}, {gamepad_read.right_stick_y})")
    print(f"Left trigger: {gamepad_read.left_trigger}")
    print(f"Right trigger: {gamepad_read.right_trigger}")
    print(f"A button: {gamepad_read.a_button}")
    print(f"B button: {gamepad_read.b_button}")
    print(f"X button: {gamepad_read.x_button}")
    print(f"Y button: {gamepad_read.y_button}")
```

## get_gamepad_summary
```python
def get_gamepad_summary(self) -> Dict[str, Any]:
```
Gets comprehensive gamepad device summary including device information and gamepad read data.

**Returns:**
- `Dict[str, Any]`: Dictionary containing:
  - `name`: Device name
  - `device_id`: Device ID
  - `gamepad_type`: Device type (SecondaryDeviceType enum value)
  - `gamepad_read`: GamepadRead protobuf message or None

Examples:
```python
summary = gamepad.get_gamepad_summary()
print(f"Device name: {summary['name']}")
print(f"Device ID: {summary['device_id']}")
print(f"Gamepad type: {summary['gamepad_type']}")
if summary['gamepad_read'] is not None:
    print(f"Left stick: ({summary['gamepad_read'].left_stick_x}, {summary['gamepad_read'].left_stick_y})")
```

## Inherited Methods

The `Gamepad` class inherits all methods from `OptionalDeviceBase`, including:

### From OptionalDeviceBase:
- `get_device_summary()` - Get device status summary (name and device_id)

**Examples:**
```python
# Get device summary
summary = gamepad.get_device_summary()
print(f"Device name: {summary['name']}")
print(f"Device ID: {summary['device_id']}")
```

## Usage Example

```python
from hex_device import HexDeviceApi
from hex_device.generated import public_api_types_pb2

# Create API instance
api = HexDeviceApi(ws_url="ws://localhost:8080")

# Find Gamepad device by device ID
gamepad = api.find_optional_device_by_id(device_id=1)

# Or find by device type
gamepad_devices = api.find_optional_device_by_robot_type(robot_type=public_api_types_pb2.SecondaryDeviceType.SdtGamepad)

if gamepad is not None:
    # Get gamepad read data
    gamepad_read = gamepad.get_gamepad_read()
    if gamepad_read is not None:
        # Read stick values
        left_x = gamepad_read.left_stick_x
        left_y = gamepad_read.left_stick_y
        right_x = gamepad_read.right_stick_x
        right_y = gamepad_read.right_stick_y
        
        # Read trigger values
        left_trigger = gamepad_read.left_trigger
        right_trigger = gamepad_read.right_trigger
        
        # Read button states
        if gamepad_read.a_button:
            print("A button pressed")
        if gamepad_read.b_button:
            print("B button pressed")
        if gamepad_read.x_button:
            print("X button pressed")
        if gamepad_read.y_button:
            print("Y button pressed")
        
        # Read bumper states
        if gamepad_read.left_bumper:
            print("Left bumper pressed")
        if gamepad_read.right_bumper:
            print("Right bumper pressed")
        
        # Read D-pad states
        if gamepad_read.dpad_up:
            print("D-pad up pressed")
        if gamepad_read.dpad_down:
            print("D-pad down pressed")
        if gamepad_read.dpad_left:
            print("D-pad left pressed")
        if gamepad_read.dpad_right:
            print("D-pad right pressed")
    
    # Get gamepad summary
    summary = gamepad.get_gamepad_summary()
    print(f"Gamepad type: {summary['gamepad_type']}")
    
    # Get device summary
    device_summary = gamepad.get_device_summary()
    print(f"Device name: {device_summary['name']}")
    print(f"Device ID: {device_summary['device_id']}")
```

