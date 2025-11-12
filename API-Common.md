**Guide：**
- [HexDeviceApi](#hexdeviceapi)
- [DeviceBase](#devicebase)
- [MotorBase](#motorbase)
- [OptionalDeviceBase](#optionaldevicebase)

# HexDeviceApi

## `__init__`
```python
def __init__(self, ws_url: str, control_hz: int = 500, enable_kcp: bool = True, local_port: int = None):
```
Creates the main HexDevice API runtime, validates the WebSocket endpoint, and launches the internal asyncio worker thread that manages device discovery and message dispatch.

Parameters:
- `ws_url`: WebSocket URL of the HexDevice server. Raises an `InvalidWSURLException` if the value is not supported.
- `control_hz`: Frequency (Hz) for internal scheduling when processing device tasks. Defaults to 500 Hz.
- `enable_kcp`: Enables the accelerated KCP transport channel when available. When `True`, the API will negotiate the UDP tunnel, otherwise all traffic stays on the base WebSocket connection.
- `local_port`: Explicit local UDP port for the KCP client. Use `None` to let the OS pick a free port automatically.

Examples:
```python
from hex_device import HexDeviceApi
api = HexDeviceApi(ws_url=args.url, control_hz=250, enable_kcp=True, local_port=52323)
```

## device_list
```python
@property
def device_list(self):
```
Returns a read-only device list containing all HEX series device class instances currently captured in the WebSocket.  

Examples:
```python
for device in api.device_list:
    if isinstance(device, Chassis):
        pass
    elif isinstance(device, Arm):
        pass
```

## optional_device_list
```python
@property
def optional_device_list(self):
```
Returns a read-only view of the optional device instances that were dynamically attached to registered primary devices (for example Hands peripherals). The list blocks all mutating methods so the SDK can keep ownership of lifecycle management.

Examples:
```python
for optional in api.optional_device_list:
    print(optional.device_id, optional.device_type)
```

## find_device_by_robot_type
```python
def find_device_by_robot_type(self, robot_type) -> Optional[DeviceBase]
```
Matches device instances in the current device list based on the provided robot type.  

Examples:
```python
# RtArmArcherD6Y = 16;
archer = api.find_device_by_robot_type(16)
```

## find_optional_device_by_id
```python
def find_optional_device_by_id(self, device_id: int) -> Optional[OptionalDeviceBase]:
```
Retrieves an optional device (secondary device) by the `device_id` reported in `SecondaryDeviceStatus`. Returns `None` if the device has not been discovered or removed.

## get_device_task_status
```python
def get_device_task_status(self) -> Dict[str, Any]:
```
Gets a high-level snapshot of the internal task scheduler. The returned dictionary contains:
- `total_devices`: Number of devices currently tracked.
- `active_tasks`: Total count of periodic tasks still running.
- `device_tasks`: Mapping keyed by device name with `device_id`, `device_type`, `robot_type`, and task state flags (`task_done`, `task_cancelled`).

## close
```python
def close(self):
```
Cleans up all asynchronous threads and closes the API interface.  
Examples:
```python
try:
    while not api.is_api_exit():
        pass
except KeyboardInterrupt:
    print("Received Ctrl-C.")
finally:
    api.close()
```

## is_api_exit
```python
def is_api_exit(self) -> bool:
```
Checks if the API has exited.  
Examples:
```python
try:
    while not api.is_api_exit():
        pass
except KeyboardInterrupt:
    print("Received Ctrl-C.")
finally:
    api.close()
```

## get_raw_data
```python
def get_raw_data(self) -> Tuple[public_api_up_pb2.APIUp, int]:
```
Returns a tuple of the oldest buffered `APIUp` protobuf message and the remaining queue length. Internally the API maintains a sliding window buffer (maximum length `RAW_DATA_LEN`, equal to 50 frames). By consuming the queue frequently you can reconstruct a lossless real-time stream; helper routines such as `_parse_wheel_data` can be used to decode the payload.
Examples:
```python
while not api.is_api_exit():
    (data, num) = api.get_raw_data()
    if data.is_some():
        # parse data
```

# DeviceBase

## `__init__`
```python
def __init__(self, name: str = "", send_message_callback=None):
```
Automatically called by HexDeviceApi, passing in the device name and callback function for sending WebSocket messages.

## set_has_new_data
```python
def set_has_new_data(self):
```
Sets the current data status to have new data.

## has_new_data
```python
def has_new_data(self) -> bool:
```
Checks if there is new data.  
Example:
```python
for device in api.device_list:
    if device.has_new_data()
        # do something for device...
        pass
```

## clear_new_data_flag
```python
def clear_new_data_flag(self):
```
Clears the new data flag.

## get_device_summary
```python
def get_device_summary(self) -> Dict[str, Any]:
```
Gets the current device status.


# MotorBase
## `__init__`
```python
def __init__(self, motor_count: int, name: str = ""):
```
Automatically called by HexDeviceApi, passing in the motor count and motor group name.

## states
```python
@property
def states(self) -> List[str]:
```
Gets the status list of all motors, with status values of "normal" or "error".  
Examples:
```python
for i, state in enumerate(motor.states):
    if state == "error":
        print(f"Motor {i} has error: {motor.error_codes[i]}")
```

## error_codes
```python
@property
def error_codes(self) -> List[Optional[int]]:
```
Gets the error code list of all motors, where None indicates no error.  
Examples:
```python
for i, error_code in enumerate(motor.error_codes):
    if error_code is not None:
        print(f"Motor {i} error code: {error_code}")
```

## positions
```python
@property
def positions(self) -> np.ndarray:
```
Gets the position array of all motors (unit: rad).  
Examples:
```python
current_positions = motor.positions
print(f"All motor positions: {current_positions}")
```

## velocities
```python
@property
def velocities(self) -> np.ndarray:
```
Gets the velocity array of all motors (unit: rad/s).  
Examples:
```python
current_velocities = motor.velocities
print(f"All motor velocities: {current_velocities}")
```

## torques
```python
@property
def torques(self) -> np.ndarray:
```
Gets the torque array of all motors (unit: Nm).  
Examples:
```python
current_torques = motor.torques
print(f"All motor torques: {current_torques}")
```

## driver_temperature
```python
@property
def driver_temperature(self) -> np.ndarray:
```
Gets the driver temperature array of all motors (unit: °C).  
Examples:
```python
temps = motor.driver_temperature
if np.any(temps > 80):
    print("Warning: Driver overheating detected")
```

## motor_temperature
```python
@property
def motor_temperature(self) -> np.ndarray:
```
Gets the motor temperature array of all motors (unit: °C).  
Examples:
```python
temps = motor.motor_temperature
if np.any(temps > 70):
    print("Warning: Motor overheating detected")
```

## voltage
```python
@property
def voltage(self) -> np.ndarray:
```
Gets the voltage array of all motors (unit: V).  
Examples:
```python
voltages = motor.voltage
print(f"Motor voltages: {voltages}")
```

## pulse_per_rotation
```python
@property
def pulse_per_rotation(self) -> np.ndarray:
```
Gets the pulse per rotation array of all motors, used for encoder position calculation.  
Examples:
```python
ppr = motor.pulse_per_rotation
print(f"Pulses per rotation: {ppr}")
```

## wheel_radius
```python
@property
def wheel_radius(self) -> np.ndarray:
```
Gets the wheel radius array of all motors (unit: m), only meaningful for motors on wheels.  
Examples:
```python
radius = motor.wheel_radius
linear_velocity = motor.velocities * radius
```

## target_positions
```python
@property
def target_positions(self) -> np.ndarray:
```
Gets the target position array of all motors (unit: rad), only valid when the current command type is POSITION.  
Examples:
```python
target_pos = motor.target_positions
current_pos = motor.positions
error = target_pos - current_pos
```

## target_velocities
```python
@property
def target_velocities(self) -> np.ndarray:
```
Gets the target velocity array of all motors (unit: rad/s), only valid when the current command type is SPEED.  
Examples:
```python
target_vel = motor.target_velocities
current_vel = motor.velocities
vel_error = target_vel - current_vel
```

## target_torques
```python
@property
def target_torques(self) -> np.ndarray:
```
Gets the target torque array of all motors (unit: Nm), only valid when the current command type is TORQUE.  
Examples:
```python
target_torque = motor.target_torques
current_torque = motor.torques
```

## has_new_data
```python
@property
def has_new_data(self) -> bool:
```
Checks if there are new motor data updates.  
Examples:
```python
if motor.has_new_data:
    # process new data
    process_motor_data(motor)
```

## get_motor_state
```python
def get_motor_state(self, motor_index: int) -> str:
```
Gets the status of the specified motor, returning "normal" or "error". Clears the new data flag after calling.  
Examples:
```python
state = motor.get_motor_state(0)
if state == "error":
    print(f"Motor 0 is in error state")
```

## get_simple_motor_status
```python
def get_simple_motor_status(self) -> Dict[str, Any]:
        """Get simple motor status"""
        with self._data_lock:
            return {
                'pos': self._positions.tolist(), //rad
                'vel': self._velocities.tolist(), //rad/s
                'eff': self._torques.tolist(), //Nm
                'ts': {
                        "s": self._last_update_time // 1_000_000_000,
                        "ns": self._last_update_time % 1_000_000_000,
                    }
            }
```
Get basic motor motion information at one time.


## get_motor_position
```python
def get_motor_position(self, motor_index: int) -> float:
```
Gets the position of the specified motor (unit: rad). Clears the new data flag after calling.  
Examples:
```python
position = motor.get_motor_position(0)
print(f"Motor 0 position: {position} rad")
```

## get_motor_positions
```python
def get_motor_positions(self) -> List[float]:
```
Gets the position list of all motors (unit: rad). Clears the new data flag after calling.  
Examples:
```python
positions = motor.get_motor_positions()
for i, pos in enumerate(positions):
    print(f"Motor {i}: {pos} rad")
```

## get_motor_velocity
```python
def get_motor_velocity(self, motor_index: int) -> float:
```
Gets the velocity of the specified motor (unit: rad/s). Clears the new data flag after calling.  
Examples:
```python
velocity = motor.get_motor_velocity(0)
print(f"Motor 0 velocity: {velocity} rad/s")
```

## get_motor_velocities
```python
def get_motor_velocities(self) -> List[float]:
```
Gets the velocity list of all motors (unit: rad/s). Clears the new data flag after calling.  
Examples:
```python
velocities = motor.get_motor_velocities()
avg_velocity = sum(velocities) / len(velocities)
```

## get_motor_torque
```python
def get_motor_torque(self, motor_index: int) -> float:
```
Gets the torque of the specified motor (unit: Nm). Clears the new data flag after calling.  
Examples:
```python
torque = motor.get_motor_torque(0)
print(f"Motor 0 torque: {torque} Nm")
```

## get_motor_torques
```python
def get_motor_torques(self) -> List[float]:
```
Gets the torque list of all motors (unit: Nm). Clears the new data flag after calling.  
Examples:
```python
torques = motor.get_motor_torques()
total_torque = sum(torques)
```

## get_motor_driver_temperature
```python
def get_motor_driver_temperature(self, motor_index: int) -> float:
```
Gets the driver temperature of the specified motor (unit: °C). Clears the new data flag after calling.  
Examples:
```python
temp = motor.get_motor_driver_temperature(0)
if temp > 80:
    print(f"Warning: Driver 0 overheating at {temp}°C")
```

## get_motor_temperature
```python
def get_motor_temperature(self, motor_index: int) -> float:
```
Gets the temperature of the specified motor (unit: °C). Clears the new data flag after calling.  
Examples:
```python
temp = motor.get_motor_temperature(0)
if temp > 70:
    print(f"Warning: Motor 0 overheating at {temp}°C")
```

## get_motor_voltage
```python
def get_motor_voltage(self, motor_index: int) -> float:
```
Gets the voltage of the specified motor (unit: V). Clears the new data flag after calling.  
Examples:
```python
voltage = motor.get_motor_voltage(0)
print(f"Motor 0 voltage: {voltage} V")
```

## get_motor_pulse_per_rotation
```python
def get_motor_pulse_per_rotation(self, motor_index: int) -> float:
```
Gets the pulse per rotation of the specified motor.  
Examples:
```python
ppr = motor.get_motor_pulse_per_rotation(0)
print(f"Motor 0 pulses per rotation: {ppr}")
```

## get_motor_wheel_radius
```python
def get_motor_wheel_radius(self, motor_index: int) -> float:
```
Gets the wheel radius of the specified motor (unit: m).  
Examples:
```python
radius = motor.get_motor_wheel_radius(0)
linear_velocity = motor.get_motor_velocity(0) * radius
```

## motor_command
```python
def motor_command(self, command_type: CommandType, values: Union[List[bool], List[float], List[MitMotorCommand], np.ndarray]):
```
Sets motor commands, supporting five command types: BRAKE, SPEED, POSITION, TORQUE, and MIT.  
Examples:
```python
# Set speed command
motor.motor_command(CommandType.SPEED, [1.0, -1.0, 0.5])

# Set position command
motor.motor_command(CommandType.POSITION, [0.0, 1.57, 3.14])

# Set brake command
motor.motor_command(CommandType.BRAKE, [True, True, False])

# Set torque command
motor.motor_command(CommandType.TORQUE, [0.5, 0.3, 0.0])

# Use numpy data
motor.motor_command(CommandType.POSITION, np.array([0.0, 1.57, 3.14]))
```

## mit_motor_command
```python
def mit_motor_command(self, mit_commands: List[MitMotorCommand]):
```
Sets MIT motor commands, with each command containing torque, speed, position, kp, and kd parameters.  
Examples:
```python
mit_cmds = [
    MitMotorCommand(torque=0.5, speed=1.0, position=0.0, kp=10.0, kd=1.0),
    MitMotorCommand(torque=0.3, speed=0.5, position=1.57, kp=8.0, kd=0.8)
]
motor.mit_motor_command(mit_cmds)
```

## update_motor_data
```python
def update_motor_data(self, positions: List[float], velocities: List[float], torques: List[float], driver_temperature: List[float], motor_temperature: List[float], voltage: List[float], pulse_per_rotation: Optional[List[float]] = None, wheel_radius: Optional[List[float]] = None, error_codes: Optional[List[Optional[int]]] = None, current_targets: Optional[List[public_api_types_pb2.SingleMotorTarget]] = None):
```
Updates all motor data, called internally by HexDeviceApi. Position data is automatically converted to radians.  
Examples:
```python
# Usually called internally by HexDeviceApi
motor.update_motor_data(
    positions=[32768, 16384, 0],  # encoder positions
    velocities=[1.0, 0.5, 0.0],
    torques=[0.5, 0.3, 0.0],
    driver_temperature=[45.0, 42.0, 40.0],
    motor_temperature=[35.0, 33.0, 30.0],
    voltage=[24.0, 24.1, 23.9]
)
```

## clear_new_data_flag
```python
def clear_new_data_flag(self):
```
Clears the new data flag, indicating that new data has been processed.  
Examples:
```python
if motor.has_new_data:
    # process data
    process_data(motor)
    motor.clear_new_data_flag()
```

## get_motor_summary
```python
def get_motor_summary(self) -> Dict[str, Any]:
```
Gets the motor group status summary, containing status information and target commands for all motors.  
Examples:
```python
summary = motor.get_motor_summary()
print(f"Motor count: {summary['motor_count']}")
print(f"States: {summary['states']}")
print(f"Positions: {summary['positions']}")
if summary['target_command']:
    print(f"Command type: {summary['target_command']['command_type']}")
```

## get_motor_status
```python
def get_motor_status(self, motor_index: int) -> Dict[str, Any]:
```
Gets detailed status information for the specified motor, including current status and target commands.  
Examples:
```python
status = motor.get_motor_status(0)
print(f"Motor 0 state: {status['state']}")
print(f"Position: {status['position']} rad")
print(f"Velocity: {status['velocity']} rad/s")
print(f"Target velocity: {status['target_velocity']} rad/s")
if status['error_code'] is not None:
    print(f"Error code: {status['error_code']}")
```

# OptionalDeviceBase

## `__init__`
```python
def __init__(self, read_only: bool, name: str = "", send_message_callback=None, device_id: int = None):
```
Initializes an optional device base class. These devices are matched by `device_id` (from `SecondaryDeviceStatus`) rather than `robot_type` and are used for processing optional fields in APIUp messages.

**Parameters:**
- `read_only` (bool): Whether the device is read-only
- `name` (str, optional): Device name, defaults to "OptionalDevice"
- `send_message_callback` (callable, optional): Callback function for sending messages
- `device_id` (int, optional): Unique identifier assigned from secondary device discovery

**Examples:**
```python
# Create a read-only optional device
device = MyOptionalDevice(read_only=True, name="IMUDevice")

# Create a read-write optional device with callback
device = MyOptionalDevice(read_only=False, name="GamepadDevice", send_message_callback=my_callback)
```

## set_has_new_data
```python
def set_has_new_data(self):
```
Sets the current data status to have new data. This method is thread-safe.

**Examples:**
```python
# Called internally when new data arrives
device.set_has_new_data()
```

## has_new_data
```python
def has_new_data(self) -> bool:
```
Checks if there is new data available. This method is thread-safe.

**Returns:**
- `bool`: True if there is new data, False otherwise

**Examples:**
```python
if device.has_new_data():
    # Process new data
    process_optional_data(device)
```

## clear_new_data_flag
```python
def clear_new_data_flag(self):
```
Clears the new data flag, indicating that new data has been processed. This method is thread-safe.

**Examples:**
```python
if device.has_new_data():
    # Process data
    process_data(device)
    device.clear_new_data_flag()
```

## get_device_summary
```python
def get_device_summary(self) -> Dict[str, Any]:
```
Gets the device status summary including name, assigned device ID, data status, and last update time (in nanoseconds).

**Returns:**
- `Dict[str, Any]`: Dictionary containing device status information

**Examples:**
```python
summary = device.get_device_summary()
print(f"Device: {summary['name']}")
print(f"Device ID: {summary['device_id']}")
print(f"Has new data: {summary['has_new_data']}")
print(f"Last update: {summary['last_update_time']}")
```
