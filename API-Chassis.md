The `Chassis` class inherits from [DeviceBase](API-Common#Devicebase) and [MotorBase](API-Common#Motorbase), primarily implementing the mapping to `BaseStatus`. This class corresponds to `BaseStatus` in the proto, managing chassis status and motor control.

Supported robot types:
- `RtArk2LrDriver`: Chassis Mark2 (2 motors)
- `RtCustomPcwVehicle`: Custom PCW vehicle (8 motors)
- `RtMaverX4`: Maver vehicle (8 motors)
- `RtTripleOmniWheelLRDriver`: Triple Omni Wheel LR Driver

# Chassis
```python
class Chassis(DeviceBase, MotorBase):
```

The common function can be found in: [DeviceBase](API-Common#Devicebase) and [MotorBase](API-Motorbase).

## `__init__`
```python
def __init__(self, motor_count: int, robot_type: int, name: str = "Chassis", control_hz: int = 500, send_message_callback=None):
```
Automatically called by HexDeviceApi to initialize the chassis device.

Examples:
```python
# Usually called internally by HexDeviceApi
from hex_device.generated import public_api_types_pb2

# For Mark2 chassis (2 motors)
chassis_mark2 = Chassis(
    motor_count=2, 
    robot_type=public_api_types_pb2.RobotType.RtArk2LrDriver,
    name="MyChassisMark2", 
    control_hz=500
)

# For PCW vehicle (8 motors)
chassis_maver = Chassis(
    motor_count=8, 
    robot_type=public_api_types_pb2.RobotType.RtPcwVehicle,
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
def get_base_state(self) -> int:
```
Gets the chassis state.

Examples:
```python
state = chassis.get_base_state()
if state == public_api_types_pb2.BaseState.BsParked:
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
def get_vehicle_speed(self) -> Tuple[float, float, float]:
```
Gets the vehicle speed (units: m/s, m/s, rad/s).

Examples:
```python
speed_x, speed_y, speed_z = chassis.get_vehicle_speed()
print(f"Vehicle speed: x={speed_x}, y={speed_y}, angular={speed_z}")
```

## get_vehicle_position
```python
def get_vehicle_position(self) -> Tuple[float, float, float]:
```
Gets the vehicle position relative to the odometry bias zero point (units: m, m, rad).

Examples:
```python
x, y, yaw = chassis.get_vehicle_position()
print(f"Vehicle position: x={x}m, y={y}m, yaw={yaw}rad")
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

Examples:
```python
id = chassis.get_arm_config()
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

Examples:
```python
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

**Note:** For Mark2 chassis (`RtArk2LrDriver`), `speed_y` is always filtered to 0 as it only supports forward/backward and rotation movements.

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
When a command times out, the chassis will automatically lock its speed at 0. You can use this function to check whether the command has timed out.  
```python
if chassis.is_timeout():
    print("timeout!!")
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

