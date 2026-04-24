# HEX Device Python Library

Welcome to [HEX Device Python Library](https://github.com/hexfellow/hex_device_python)! This is a Python library for controlling HEX series robot devices, providing a unified API interface to manage and control various robot hardware devices.

## 📚 Documentation Navigation

- **[Installation Guide](Installation-Guide.md)** - Detailed installation steps and environment configuration
- **[Function Details](API-List.md)** - Complete API function documentation
- **[Change Log](Change-Log.md)** - Record the modifications and development plans of each version

## 🚀 Quick Start

### Installation

Please refer to our [Installation Guide](Installation-Guide.md) for detailed installation steps.

### Basic Usage

Please checkout the [test/main.py](https://github.com/hexfellow/hex_device_python/blob/main/tests/main.py)

## 🔧 Main Features

### Device Management
- **Unified Interface**: Provides unified device management interface through `HexDeviceApi`
- **Device Discovery**: Automatically detects and manages all HEX series devices in WebSocket connections
- **Real-time Data**: Supports high-frequency (default 500Hz) real-time data updates and control
- **Status Monitoring**: Real-time monitoring of device status, battery information, warning messages, etc.

### Chassis Control
- **Chassis**: Supports chassis control for multiple robot types (Mark2, Maver, PCW vehicles, etc.)
- **Speed Control**: Supports XYZ three-axis speed control, suitable for omnidirectional mobile platforms
- **Motor Control**: Direct motor-level control
- **Odometry**: Built-in odometry functionality providing position and velocity feedback

### Motor System
- **Multiple Control Modes**: Supports BRAKE, SPEED, POSITION, TORQUE, MIT five motor control modes (if hardware supports)
- **Real-time Feedback**: Obtains real-time status of motor position, velocity, torque, temperature, voltage, etc.
- **Safety Monitoring**: Built-in motor status monitoring and error detection mechanisms


## 🎯 Supported Device Types

Currently supports the following robot types:

**Chassis:**
- `RtArk2Lr1`: Ark2 vehicle
- `RtMaverX4D`: Maver X4D chassis
- `RtMaverL4D`: Maver L4D chassis
- `RtTriggerA3Lr1`: Trigger A3LR1 chassis

**Arm:**
- `RtArmSaberD6x`: Saber 6-DOF arm
- `RtArmSaberD7x`: Saber 7-DOF arm
- `RtArmArcherD6Y_P1`: Archer 6-DOF arm
- `RtArmArcherY6L_V1`: Archer long arm
- `RtArmArcherY6_H1`: Archer high-load arm
- `RtArmFireflyY6_H1`: Firefly high-speed arm
- `RtHelloArcherY6_H1`: Hello Archer collaborative arm
- `RtHelloFireflyY6_H1`: Hello Firefly collaborative arm
- `RtArmArcherX7h1`: Archer 7-DOF high-precision arm

**Linear Lift:**
- `RtIotaP1`: Iota P1 lift
- `RtIotaVc1`: Iota VC1 lift

**Zeta Lift:**
- `RtZetaVc2`: Zeta VC2 rotating lift

**Hands:**
- `SdtHandGp100`: GP100 hand
- `SdtHandGp80G1`: GP80G1 hand
- `SdtHandGr100`: GR100 hand

**Optional Devices:**
- `SdtGamepad`: Gamepad controller
- `SdtImuY200`: IMU sensor

## 🤝 Getting Help

If you encounter problems during usage:

1. Check the [Function Details Documentation](API-List.md) to understand API usage
2. Ensure you are using **public** function interfaces
3. Check device connections and WebSocket configuration
4. Create an `issue`

**Note**: This documentation only records **public** function interfaces of all classes. If you modify the `hex_device_python` library or call **non-public** functions, problems arising from this will not be supported.

---

Start your robotic control journey! 🤖
