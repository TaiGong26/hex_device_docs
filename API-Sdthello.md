The `SdtHello` class inherits from [OptionalDeviceBase](API-Common#OptionalDeviceBase), primarily implementing Hello device data reading and RGB stripe control. This class processes the optional `hello1j1t4b_status` field from APIUp messages.

Supported device types:
- `SdtHello1J1T4BV1`: Hello1J1T4B V1 device type

# SdtHello
```python
class SdtHello(OptionalDeviceBase):
```

The common function can be found in: [OptionalDeviceBase](API-Common#OptionalDeviceBase).

## `__init__`
```python
def __init__(self, device_id, device_type, send_message_callback, name: str = "SdtHello", control_hz: int = 500, read_only: bool = False):
```
Automatically called by HexDeviceApi to initialize the SdtHello device.

**Parameters:**
- `device_id`: Device ID from SecondaryDeviceStatus
- `device_type`: Device type (SecondaryDeviceType enum, e.g., SdtHello1J1T4BV1)
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
  - `pos`: List of position values [trigger, joystick_x, joystick_y, btn_a, btn_b, btn_x, btn_y]
    - `trigger`: Trigger value (float)
    - `joystick_x`: Joystick X axis value (float)
    - `joystick_y`: Joystick Y axis value (float)
    - `btn_a`, `btn_b`, `btn_x`, `btn_y`: Button states (1.0 for pressed, -1.0 for not pressed)
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
    btn_a = status['pos'][3]
    print(f"Trigger: {trigger}, Joystick: ({joystick_x}, {joystick_y}), Button A: {btn_a}")
```

## set_rgb_stripe_command
```python
def set_rgb_stripe_command(self, r: list[int], g: list[int], b: list[int]):
```
Sets RGB stripe command to control the LED colors on the Hello device.

**Parameters:**
- `r` (list[int]): List of red values (0-255) for each LED
- `g` (list[int]): List of green values (0-255) for each LED
- `b` (list[int]): List of blue values (0-255) for each LED

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

The `SdtHello` class inherits all methods from `OptionalDeviceBase`, including:

### From OptionalDeviceBase:
- `get_device_summary()` - Get device status summary (name and device_id)

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
        btn_a = status['pos'][3]
        btn_b = status['pos'][4]
        btn_x = status['pos'][5]
        btn_y = status['pos'][6]
        
        print(f"Trigger: {trigger}")
        print(f"Joystick: ({joystick_x}, {joystick_y})")
        print(f"Buttons: A={btn_a}, B={btn_b}, X={btn_x}, Y={btn_y}")
    
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
