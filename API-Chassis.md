# Chassis API Documentation
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
5. [Control Methods](#control-methods)
   - [start](#start)
   - [stop](#stop)
   - [enable](#enable)
   - [disable](#disable)
6. [Odometry Methods](#odometry-methods)
   - [clear_odom_bias](#clear_odom_bias)
   - [get_vehicle_speed](#get_vehicle_speed)
   - [get_vehicle_position](#get_vehicle_position)
7. [Status Methods](#status-methods)
   - [get_base_state](#get_base_state)
   - [is_api_control_initialized](#is_api_control_initialized)
   - [get_battery_info](#get_battery_info)
   - [get_parking_stop_detail](#get_parking_stop_detail)
   - [get_warning](#get_warning)
   - [get_session_holder](#get_session_holder)
   - [get_my_session_id](#get_my_session_id)
   - [is_timeout](#is_timeout)
   - [get_status_summary](#get_status_summary)
8. [Command Methods](#command-methods)
   - [motor_command](#motor_command)
   - [set_vehicle_speed](#set_vehicle_speed)
   - [clear_parking_stop](#clear_parking_stop)


## Overview

The `Chassis` class inherits from [DeviceBase](API-Common#Devicebase) and [MotorBase](API-Common#Motorbase), primarily implementing the mapping to `BaseStatus`. This class corresponds to `BaseStatus` in the proto, managing chassis status and motor control.

## Supported Robot Types

| Robot Type | Motor Count | Description | Use Case |
|------------|-------------|-------------|----------|
| `RtTriggerA3Lr1` | - | Trigger A3 LR1 chassis | - |
| `RtMaverX4D` | - | Maver X4D chassis | - |
| `RtMaverL4D` | - | Maver L4D chassis | - |
| `RtArk2Lr1` | - | Ark2 LR1 chassis | - |

## Class Definition
```python
class Chassis(DeviceBase, MotorBase):
```

The common function can be found in: [DeviceBase](API-Common#Devicebase) and [MotorBase](API-Motorbase).

## `__init__`
```python
def __init__(self, motor_count: int, robot_type: int, proto_version: tuple[int, int], name: str = "Chassis", control_hz: int = 500, send_message_callback=None):
```
Automatically called by HexDeviceApi to initialize the chassis device.

**Parameters:**
- `motor_count` (int): Number of motors
- `robot_type` (int): Robot type ID from `public_api_types_pb2.RobotType`
- `proto_version` (tuple[int, int]): Protocol version (major, minor)
- `name` (str, optional): Device name. Defaults to "Chassis"
- `control_hz` (int, optional): Control frequency in Hz. Defaults to 500
- `send_message_callback` (optional): Callback function for sending messages

Examples:
```python
# Usually called internally by HexDeviceApi
from hex_device.generated import public_api_types_pb2

# For Ark2 chassis (2 motors)
chassis_ark2 = Chassis(
    motor_count=2, 
    robot_type=public_api_types_pb2.RobotType.RtArk2Lr1,
    proto_version=(1, 0),
    name="MyChassisArk2", 
    control_hz=500
)

# For Maver X4D chassis (8 motors)
chassis_maver = Chassis(
    motor_count=8, 
    robot_type=public_api_types_pb2.RobotType.RtMaverX4D,
    proto_version=(1, 0),
    name="MyChassisMaver", 
    control_hz=500
)
```

## start
```python
def start(self):
```
Sends initialization command and sets `api_control_initialized` to `True`. The chassis will only start responding to control commands after calling this function.  
**Note:** If there is already a controller, this command will be ignored by the chassis until control is released.

Examples:
```python
chassis.start()
print("Chassis control started")
```

## stop
```python
def stop(self):
```
Sends stop command and sets `api_control_initialized` to `False`. After calling this function, the chassis will enter Disable mode and stop connection monitoring and command listening. This is the correct way to disconnect. If you exit the program directly without calling stop, the robotic arm will enter a connection timeout error state.

Examples:
```python
chassis.stop()
print("Chassis control stopped")
```

## clear_odom_bias
```python
def clear_odom_bias(self):
```
Resets the odometry position bias, setting the current position as the origin.

Examples:
```python
chassis.clear_odom_bias()
print("Odometry bias cleared")
```

## get_base_state
```python
def get_base_state(self) -> str:
```
Gets the chassis state as a string name (e.g., "BsParked", "BsDriving").

**Returns:**
- `str`: The chassis state name

Examples:
```python
state = chassis.get_base_state()
if state == "BsParked":
    print("Chassis is parked")
```

## is_api_control_initialized
```python
def is_api_control_initialized(self) -> bool:
```
Checks if API control is initialized.

Examples:
```python
if chassis.is_api_control_initialized():
    # Can start sending control commands
    chassis.set_vehicle_speed(1.0, 0.0, 0.0)
```

## get_battery_info
```python
def get_battery_info(self) -> Dict[str, Any]:
```
Gets battery information, including voltage, charge level per thousand, and charging status.

Examples:
```python
battery = chassis.get_battery_info()
print(f"Battery voltage: {battery['voltage']}V")
print(f"Battery level: {battery['thousandth']/10}%")
if battery['charging']:
    print("Battery is charging")
```

## get_vehicle_speed
```python
def get_vehicle_speed(self, pop: bool = True) -> Optional[Tuple[float, float, float]]:
```
Gets the vehicle speed (units: m/s, m/s, rad/s).

**Parameters:**
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

**Returns:**
- `Optional[Tuple[float, float, float]]`: Tuple of (speed_x, speed_y, speed_z) or None if queue is empty

Examples:
```python
# Get latest speed (pops from queue)
speed_x, speed_y, speed_z = chassis.get_vehicle_speed()
print(f"Vehicle speed: x={speed_x}, y={speed_y}, angular={speed_z}")

# Peek at latest speed without popping
speed_x, speed_y, speed_z = chassis.get_vehicle_speed(pop=False)
print(f"Current speed (not consumed): x={speed_x}, y={speed_y}, angular={speed_z}")
```

## get_vehicle_position
```python
def get_vehicle_position(self, pop: bool = True) -> Optional[Tuple[float, float, float]]:
```
Gets the vehicle position relative to the odometry bias zero point (units: m, m, rad).

**Parameters:**
- `pop` (bool, optional): If True, pops from queue (FIFO). If False, reads latest data without popping. Defaults to True

**Returns:**
- `Optional[Tuple[float, float, float]]`: Tuple of (x, y, yaw) or None if queue is empty

Examples:
```python
# Get latest position (pops from queue)
x, y, yaw = chassis.get_vehicle_position()
print(f"Vehicle position: x={x}m, y={y}m, yaw={yaw}rad")

# Peek at latest position without popping
x, y, yaw = chassis.get_vehicle_position(pop=False)
print(f"Current position (not consumed): x={x}m, y={y}m, yaw={yaw}rad")
```

## get_parking_stop_detail
```python
def get_parking_stop_detail(self):
```
Gets the detailed reason for vehicle stop.

Examples:
```python
stop_detail = chassis.get_parking_stop_detail()
if stop_detail:
    print(f"Parking stop reason: {stop_detail.reason}")
```

## get_warning
```python
def get_warning(self) -> Optional[int]:
```
Gets warning information.

Examples:
```python
warning = chassis.get_warning()
if warning is not None:
    warning_name = public_api_types_pb2.WarningCategory.Name(warning)
    print(f"Warning: {warning_name}")
```

## get_session_holder
```python
def get_session_holder(self) -> int:
```
Gets the session ID of the current controller. Returns 0 when no one is controlling the robotic arm.

**Returns:**
- `int`: Session ID of the current controller, 0 if no one is controlling

Examples:
```python
id = chassis.get_session_holder()
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
id = chassis.get_my_session_id()
print(f"My session id is {id}")
```

## clear_parking_stop
```python
def clear_parking_stop(self):
```
Try to clear parking stop.The error can only be cleared in part.

Examples:
```python
chassis.clear_parking_stop()
```

## enable
```python
def enable(self):
```
Enables the chassis, canceling zero impedance mode.

Examples:
```python
chassis.enable()
print("Chassis enabled")
```

## disable
```python
def disable(self):
```
Disables the chassis, setting it to zero impedance mode.

Examples:
```python
chassis.disable()
print("Chassis disabled (zero resistance)")
```

## motor_command
```python
def motor_command(self, command_type: CommandType, values: List[float]):
```
Sets chassis motor commands, only available in non-simple control mode.

**Parameters:**
- `command_type` (CommandType): Type of command (BRAKE, SPEED, POSITION, TORQUE, MIT)
- `values` (List[float]): Command values

Examples:
```python
from hex_device.motor_base import CommandType

# For Mark2 (2 motors)
chassis.motor_command(CommandType.SPEED, [1.0, 1.0])
chassis.motor_command(CommandType.BRAKE, [True, True])

# For Maver (8 motors)
chassis.motor_command(CommandType.SPEED, [1.0, 1.0, 1.0, 1.0, 0.0, 0.0, 0.0, 0.0])
chassis.motor_command(CommandType.BRAKE, [True, True, True, True, True, True, True, True])
```

## set_vehicle_speed
```python
def set_vehicle_speed(self, speed_x: float, speed_y: float, speed_z: float):
```
Sets the vehicle XYZ speed, only available in simple control mode. 

**Note:** For Ark2 chassis (`RtArk2Lr1`), `speed_y` is always filtered to 0 as it only supports forward/backward and rotation movements.

Examples:
```python
# Set to move forward at 1m/s
chassis.set_vehicle_speed(1.0, 0.0, 0.0)

# Set to move left at 0.5m/s (Maver only, ignored for Mark2)
chassis.set_vehicle_speed(0.0, 0.5, 0.0)

# Set to turn with 0.5 rad/s angular velocity
chassis.set_vehicle_speed(0.0, 0.0, 0.5)

# Stop
chassis.set_vehicle_speed(0.0, 0.0, 0.0)
```

## is_timeout
```python
def is_timeout(self) -> bool:
```
Checks if the command has timed out. When a command times out, the chassis will automatically lock its speed at 0.

**Returns:**
- `bool`: True if the command has timed out, False otherwise

**Notes:**
- The timeout threshold is 100ms
- The chassis will automatically stop when a timeout occurs

Examples:
```python
if chassis.is_timeout():
    print("Command timeout detected!")
    # Take appropriate action, e.g., re-send command
    chassis.set_vehicle_speed(0.0, 0.0, 0.0)  # Stop the chassis
```

## get_status_summary
```python
def get_status_summary(self) -> Dict[str, Any]:
```
Gets the complete chassis status summary, including device status, motor status, chassis status, battery information, etc.

Examples:
```python
summary = chassis.get_status_summary()
print(f"Base state: {summary['base_state']}")
print(f"Motor count: {summary['motor_count']}")
print(f"Battery voltage: {summary['battery_info']['voltage']}V")
print(f"Vehicle position: {summary['vehicle_position']}")
```

<!-- 
## Best Practices

1. **Always call `stop()` before exiting**
   - Ensures proper disconnection and prevents timeout errors

2. **Maintain consistent command rate**
   - Send commands at least every 50ms to avoid timeouts
   - The chassis has a 100ms timeout threshold

3. **Check session holder before starting**
   - Use `get_session_holder()` to check if another controller is active

4. **Reset odometry bias when needed**
   - Use `clear_odom_bias()` to set current position as origin

5. **Optimize data retrieval**
   - Use `pop=True` when processing data in order
   - Use `pop=False` when you only need the latest value

## Troubleshooting

1. **Command Timeout**
   - **Symptom**: `is_timeout()` returns True
   - **Solution**: Increase command sending frequency, check network connection

2. **Cannot Start Control**
   - **Symptom**: `start()` has no effect
   - **Solution**: Wait for other controller to release control or restart chassis

3. **Vehicle Not Moving**
   - **Symptom**: `set_vehicle_speed()` called but vehicle doesn't move
   - **Solution**: Call `start()` and `enable()`, check speed values

4. **Inaccurate Odometry**
   - **Symptom**: Position values drift over time
   - **Solution**: Call `clear_odom_bias()` periodically

5. **Debugging Tips**
   - Enable verbose logging to see detailed communication
   - Regularly check `get_base_state()` and `get_status_summary()`
   - Test with simple commands before complex operations

 -->