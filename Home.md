# HEX Device Python Library

Welcome to [HEX Device Python Library](https://github.com/hexfellow/hex_device_python)! This is a Python library for controlling HEX series robot devices, providing a unified API interface to manage and control various robot hardware devices.

## 🚀 Quick Start

### Installation

Please refer to our [Installation Guide](Installation-Guide) for detailed installation steps.

### Basic Usage

```python
from hex_device import HexDeviceApi

# Create API instance
api = HexDeviceApi(ws_url="ws://192.168.1.1:8439", control_hz=250)

try:
    # Get device list
    for device in api.device_list:
        if isinstance(device, ChassisMaver):
            # Enable chassis
            device.enable()
            # Set vehicle speed
            device.set_vehicle_speed(1.0, 0.0, 0.0)
        
    # Main loop
    while not api.is_api_exit():
        # Process device data
        for device in api.device_list:
            if device.has_new_data():
                # Handle new data
                pass
                
except KeyboardInterrupt:
    print("Received Ctrl-C.")
finally:
    api.close()
```

## 🔧 Main Features

### Device Management
- **Unified Interface**: Provides unified device management interface through `HexDeviceApi`
- **Device Discovery**: Automatically detects and manages all HEX series devices in WebSocket connections
- **Real-time Data**: Supports high-frequency (default 500Hz) real-time data updates and control
- **Status Monitoring**: Real-time monitoring of device status, battery information, warning messages, etc.

### Chassis Control
- **ChassisMaver**: Supports chassis control for PCW vehicles and custom PCW vehicles
- **Speed Control**: Supports XYZ three-axis speed control, suitable for omnidirectional mobile platforms
- **Motor Control**: Direct motor-level control
- **Odometry**: Built-in odometry functionality providing position and velocity feedback

### Motor System
- **Multiple Control Modes**: Supports BRAKE, SPEED, POSITION, TORQUE, MIT five motor control modes (if hardware supports)
- **Real-time Feedback**: Obtains real-time status of motor position, velocity, torque, temperature, voltage, etc.
- **Safety Monitoring**: Built-in motor status monitoring and error detection mechanisms

## 📚 Documentation Navigation

- **[Installation Guide](Installation-Guide)** - Detailed installation steps and environment configuration
- **[Function Details](API-List)** - Complete API function documentation

## 🎯 Supported Device Types

Currently supports the following robot types:
- **RtCustomPcwVehicle**: Custom PCW vehicle
- **RtPcwVehicle**: PCW vehicle
- **RtArmArcherD6Y**: Archer robotic arm

## 🤝 Getting Help

If you encounter problems during usage:

1. Check the [Function Details Documentation](API-List) to understand API usage
2. Ensure you are using **public** function interfaces
3. Check device connections and WebSocket configuration
4. Create an `issue`

**Note**: This documentation only records **public** function interfaces of all classes. If you modify the `hex_device_python` library or call **non-public** functions, problems arising from this will not be supported.

---

Start your robotic control journey! 🤖
