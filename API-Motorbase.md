# MotorBase

## 概述
MotorBase是带电机设备的基类，为底盘、机械臂等设备提供通用的电机控制功能。

## `__init__`
```python
def __init__(self, motor_count: int, name: str = ""):
```
Automatically called by HexDeviceApi, passing in the motor count and motor group name.

**参数：**
- `motor_count` (int): 电机数量
- `name` (str, optional): 电机组名称，默认为空字符串

## target_positions
```python
@property
def target_positions(self) -> np.ndarray:
```
Retrieve all the commands currently sent by the motors(rad).
Examples:
```python
print(device.target_positions)
```

## target_velocities
```python
@property
def target_velocities(self) -> np.ndarray:
```
Retrieve all the commands currently sent by the motors(rad/s).
Examples:
```python
target_velocities = motor.target_velocities
print(f"All motor target_velocities: {target_velocities}")
```

## target_torques
```python
@property
def target_torques(self) -> np.ndarray:
```
Retrieve all the commands currently sent by the motors(Nm).  
Examples:
```python
target_torques = motor.target_torques
print(f"All motor target_torques: {target_torques}")
```

## has_new_data
```python
def has_new_data(self) -> bool:
```
Checks if data queue is empty.
Example:
```python
for device in api.device_list:
    if device.has_new_data()
        # do something for device...
        pass
```

## get_motor_error_codes
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

## get_motor_state
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

## get_motor_states
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

## get_simple_motor_status
```python
def get_simple_motor_status(self, pop: bool = True) -> Optional[Dict[str, Any]]:
```
Gets simple motor status dictionary or None if queue is empty. If not pop, always returns the latest frame of data. The dictionary contains 'pos' (positions), 'vel' (velocities), 'eff' (torques), and 'ts' (timestamp).  
Examples:
```python
status = device.get_simple_motor_status()
if status is not None:
    print(f"Simple motor status: {status}")
```

## get_motor_position
```python
def get_motor_position(self, motor_index: int, pop: bool = True) -> Optional[float]:
```
Gets the position of the specified motor (unit: rad) or None if queue is empty. If not pop, always returns the latest frame of data.  
Examples:
```python
position = motor.get_motor_position(0)
if position is not None:
    print(f"Motor 0 position: {position} rad")
```

## get_motor_positions
```python
def get_motor_positions(self, pop: bool = True) -> Optional[List[float]]:
```
Gets the position list of all motors (unit: rad) or None if queue is empty. If not pop, always returns the latest frame of data.  
Examples:
```python
positions = motor.get_motor_positions()
if positions is not None:
    for i, pos in enumerate(positions):
        print(f"Motor {i}: {pos} rad")
```

## get_motor_encoder_positions
```python
def get_motor_encoder_positions(self, pop: bool = True) -> Optional[np.ndarray]:
```
Gets the encoder position list of all motors (unit: pulse) or None if queue is empty. If not pop, always returns the latest frame of data.  
Examples:
```python
positions = motor.get_motor_encoder_positions()
if positions is not None:
    for i, pos in enumerate(positions):
        print(f"Motor {i}: {pos} pulse")
```

## get_encoders_to_zero
```python
def get_encoders_to_zero(self, pop: bool = True) -> Optional[List[float]]:
```
Retrieves the encoder values from the current position to the zero point, which can be used to check the difference between the current position and the software's zero position. Note that this value is only meaningful for the robotic arm. If not pop, always returns the latest frame of data.  
Examples:
```python
encoders_bias = device.get_encoders_to_zero()
if encoders_bias is not None:
    print(f"Encoders to zero: {encoders_bias}")
```

## get_motor_velocity
```python
def get_motor_velocity(self, motor_index: int, pop: bool = True) -> Optional[float]:
```
Gets the velocity of the specified motor (unit: rad/s). If not pop, always returns the latest frame of data.  
Examples:
```python
velocity = motor.get_motor_velocity(0)
if velocity is not None:
    print(f"Motor 0 velocity: {velocity} rad/s")
```

## get_motor_velocities
```python
def get_motor_velocities(self, pop: bool = True) -> Optional[List[float]]:
```
Gets the velocity list of all motors (unit: rad/s). If not pop, always returns the latest frame of data.  
Examples:
```python
velocities = motor.get_motor_velocities()
if velocities is not None:
    avg_velocity = sum(velocities) / len(velocities)
    print(f"Average velocity: {avg_velocity} rad/s")
```

## get_motor_torque
```python
def get_motor_torque(self, motor_index: int, pop: bool = True) -> Optional[float]:
```
Gets the torque of the specified motor (unit: Nm). If not pop, always returns the latest frame of data.  
Examples:
```python
torque = motor.get_motor_torque(0)
if torque is not None:
    print(f"Motor 0 torque: {torque} Nm")
```

## get_motor_torques
```python
def get_motor_torques(self, pop: bool = True) -> Optional[List[float]]:
```
Gets the torque list of all motors (unit: Nm). If not pop, always returns the latest frame of data.  
Examples:
```python
torques = motor.get_motor_torques()
if torques is not None:
    total_torque = sum(torques)
    print(f"Total torque: {total_torque} Nm")
```

## get_motor_driver_temperatures
```python
def get_motor_driver_temperatures(self) -> Optional[np.ndarray]:
```
Gets the driver temperature list of all motors (unit: °C). Always returns the latest frame of data.  
Examples:
```python
temps = motor.get_motor_driver_temperatures()
if temps is not None:
    for i, temp in enumerate(temps):
        if temp > 80:
            print(f"Warning: Driver {i} overheating at {temp}°C")
```

## get_motor_driver_temperature
```python
def get_motor_driver_temperature(self, motor_index: int) -> Optional[float]:
```
Gets the driver temperature of the specified motor (unit: °C). Always returns the latest frame of data.  
Examples:
```python
temp = motor.get_motor_driver_temperature(0)
if temp is not None and temp > 80:
    print(f"Warning: Driver 0 overheating at {temp}°C")
```

## get_motor_temperatures
```python
def get_motor_temperatures(self) -> Optional[np.ndarray]:
```
Gets the temperature list of all motors (unit: °C). Always returns the latest frame of data.  
Examples:
```python
temps = motor.get_motor_temperatures()
if temps is not None:
    for i, temp in enumerate(temps):
        if temp > 70:
            print(f"Warning: Motor {i} overheating at {temp}°C")
```

## get_motor_temperature
```python
def get_motor_temperature(self, motor_index: int) -> Optional[float]:
```
Gets the temperature of the specified motor (unit: °C). Always returns the latest frame of data.  
Examples:
```python
temp = motor.get_motor_temperature(0)
if temp is not None and temp > 70:
    print(f"Warning: Motor 0 overheating at {temp}°C")
```

## get_motor_voltage
```python
def get_motor_voltage(self, motor_index: int) -> Optional[float]:
```
Gets the voltage of the specified motor (unit: V). Always returns the latest frame of data.  
Examples:
```python
voltage = motor.get_motor_voltage(0)
if voltage is not None:
    print(f"Motor 0 voltage: {voltage} V")
```

## get_motor_voltages
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

## get_motor_pulse_per_rotation
```python
def get_motor_pulse_per_rotation(self, motor_index: int) -> Optional[float]:
```
Gets the pulse per rotation of the specified motor. Returns None if not set.  
Examples:
```python
ppr = motor.get_motor_pulse_per_rotation(0)
if ppr is not None:
    print(f"Motor 0 pulses per rotation: {ppr}")
```

## get_motor_pulse_per_rotations
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

## get_motor_wheel_radius
```python
def get_motor_wheel_radius(self, motor_index: int) -> Optional[float]:
```
Gets the wheel radius of the specified motor (unit: m). Returns None if not set.  
Examples:
```python
radius = motor.get_motor_wheel_radius(0)
if radius is not None:
    linear_velocity = motor.get_motor_velocity(0) * radius
```

## get_motor_wheel_radii
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
mit_commands = motors.construct_mit_command(
    np.array([-0.3, -1.48, 2.86, 0.0, 0.0, 0.0]), 
    np.array([0.0, 0.0, 0.0, 0.0, 0.0, 0.0]), 
    np.array([0.0, 0.0, 0.0, 0.0, 0.0, 0.0]), 
    np.array([150.0, 150.0, 150.0, 150.0, 39.0, 39.0]), 
    np.array([12.0, 12.0, 12.0, 12.0, 0.8, 0.8])
)

# Using lists
mit_commands = motors.construct_mit_command(
    [0.3, -1.48, 2.86, 0.0, 0.0, 0.0], 
    [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], 
    [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], 
    [150.0, 150.0, 150.0, 150.0, 39.0, 39.0], 
    [12.0, 12.0, 12.0, 12.0, 0.8, 0.8]
)

# Use with motor_command
motors.motor_command(CommandType.MIT, mit_commands)
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

## get_motor_summary
```python
def get_motor_summary(self) -> Optional[Dict[str, Any]]:
```
Gets the motor group status summary, containing status information and target commands for all motors. Returns None if no data available.  
Examples:
```python
summary = motor.get_motor_summary()
if summary is not None:
    print(f"Motor count: {summary['motor_count']}")
    print(f"States: {summary['states']}")
    print(f"Positions: {summary['positions']}")
    if summary['target_command']:
        print(f"Command type: {summary['target_command']['command_type']}")
```

## get_motor_status
```python
def get_motor_status(self, motor_index: int, pop: bool = True) -> Optional[Dict[str, Any]]:
```
Gets detailed status information for the specified motor, including current status and target commands. If not pop, always returns the latest frame of data. Returns None if queue is empty.  
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

## flush_motor_data
```python
def flush_motor_data(self):
```
Clears all motor data queues in MotorBase. This method removes all data from all queues.  
Examples:
```python
motor.flush_motor_data()
```
<!-- 
## 电机参数说明

### 通用电机参数范围

| 参数 | 单位 | 范围 | 说明 |
|------|------|------|------|
| 位置 | rad | 取决于电机类型 | 电机旋转角度 |
| 速度 | rad/s | -10.0 到 10.0 | 电机旋转速度 |
| 力矩 | Nm | 取决于电机型号 | 电机输出力矩 |
| 温度 | °C | 0 到 85 | 电机温度 |
| 电压 | V | 20.0 到 28.0 | 电机工作电压 |
| 电流 | A | 0 到 20 | 电机工作电流 |

### 不同设备类型的电机参数

#### 底盘电机
- **速度范围**: -10.0 到 10.0 (rad/s)
- **电流范围**: 0 到 20 (A)
- **温度范围**: 0 到 85 (°C)
- **典型应用**: 移动机器人、AGV

#### 机械臂电机
- **位置范围**: 取决于关节类型
- **速度范围**: -5.0 到 5.0 (rad/s)
- **电流范围**: 0 到 15 (A)
- **温度范围**: 0 到 80 (°C)
- **典型应用**: 工业机械臂、协作机器人

## 电机控制示例

### 基础电机控制
```python
# 速度控制
motor.motor_command(CommandType.SPEED, [1.0, -1.0, 0.5, 0.5])

# 位置控制
motor.motor_command(CommandType.POSITION, [0.0, 1.57, 3.14, 0.0])

# 力矩控制
motor.motor_command(CommandType.TORQUE, [0.5, 0.3, 0.2, 0.0])

# 刹车控制
motor.motor_command(CommandType.BRAKE, [True, True, True, True])
```

### MIT控制示例
```python
# 构造MIT命令
mit_commands = motors.construct_mit_command(
    pos=[0.0, 1.57, 3.14, 0.0],  # 目标位置
    speed=[0.0, 0.0, 0.0, 0.0],  # 目标速度
    torque=[0.0, 0.0, 0.0, 0.0],  # 目标力矩
    kp=[150.0, 150.0, 150.0, 100.0],  # 比例增益
    kd=[12.0, 12.0, 12.0, 8.0]  # 微分增益
)

# 发送MIT命令
motors.motor_command(CommandType.MIT, mit_commands)
```

### 电机状态监控
```python
def monitor_motor_health(motor):
    """监控电机健康状态"""
    while True:
        # 获取电机温度
        temps = motor.get_motor_temperatures()
        if temps is not None:
            for i, temp in enumerate(temps):
                if temp > 70:
                    print(f"警告: 电机 {i} 温度过高: {temp}°C")
                elif temp > 60:
                    print(f"提示: 电机 {i} 温度较高: {temp}°C")
        
        # 获取电机错误代码
        error_codes = motor.get_motor_error_codes()
        if error_codes is not None:
            for i, code in enumerate(error_codes):
                if code is not None:
                    print(f"错误: 电机 {i} 错误代码: {code}")
        
        # 等待一段时间
        import time
        time.sleep(1)

# 启动监控线程
import threading
monitor_thread = threading.Thread(target=monitor_motor_health, args=(motor,))
monitor_thread.daemon = True
monitor_thread.start()
``` 
-->