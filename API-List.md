# API Overview

**Note**: This document only records the **public** function interfaces of all classes. If you modify the `hex_device_python` library or call **non-public** functions, any resulting issues will not be supported.

## System Architecture
- Device communication and thread management are provided by the `HexDeviceApi` class, which is the unified interface for using HexDevice.
- The system provides two core base classes: `DeviceBase` and `MotorBase`. All device classes inherit from `DeviceBase`, while device classes with motors inherit from `MotorBase`, implementing common interfaces between devices.

## Interface Classification

### Common Interfaces
- **[HexDeviceApi](API-Common#HexDeviceApi)** - Core API interface responsible for device discovery and message dispatch
  - Device Management: Device list retrieval, device lookup
  - Connection Management: WebSocket connection, KCP acceleration
  - Message Processing: Raw data retrieval, task status monitoring
  - Stream Mode: Direct data processing without WebSocket/KCP
- **[DeviceBase](API-Common#Devicebase)** - Device base class providing common device operations
  - Basic Control: Start/stop control
  - Status Management: Device status retrieval
- **[MotorBase](API-Common#motorbase)** - Motor base class providing common motor control
  - Motor Control: Speed control, position control
  - Status Monitoring: Motor status retrieval
- **[OptionalDeviceBase](API-Common#OptionalDeviceBase)** - Optional device base class
  - Auxiliary Device Management: Device status retrieval

### Chassis Devices
- **[Chassis](API-Chassis)** - Chassis control supporting multiple robot types
  - Motion Control: Speed control, motor control
  - Status Monitoring: Odometry, battery information, safety status
  - Advanced Features: Multiple robot type support, zero impedance mode
  - Safety Features: Parking stop, timeout detection

### Arm Devices
- **[Arm](API-Arm)** - Arm control supporting multiple arm types
  - Joint Control: Position control, speed control
  - End Effector: Position and orientation control
  - Advanced Features: Free drag, gravity compensation, MIT control
  - Configuration Management: Joint limit validation, configuration reload
  - Safety Features: Collision detection, emergency stop

### Hand Devices
- **[Hands](API-Hands)** - Hand device control
  - Grasp Control: Open/close control, force control
  - Status Monitoring: Hand status retrieval
- **[SdtHello](API-Sdthello)** - SDT Hello device control
  - Basic Control: Device start/stop
  - Status Monitoring: Device status retrieval

### Lift Mechanisms
- **[Linear](API-Linear)** - Linear lift mechanism control
  - Position Control: Target position setting
  - Status Monitoring: Current position retrieval
- **[Zeta](API-Zeta)** - Zeta lift mechanism control
  - Position Control: Target position setting
  - Status Monitoring: Current position retrieval

### Other Devices
- **[Imu](API-Imu)** - Inertial Measurement Unit
  - Data Acquisition: Acceleration, angular velocity, attitude
  - Data Parsing: Raw data processing
- **[Gamepad](API-Gamepad)** - Gamepad
  - Input Processing: Button status, joystick position
  - Event Monitoring: Input event handling

## Quick Start Guide

### Basic Usage
1. **Initialize API**: Create a HexDeviceApi instance with WebSocket URL
2. **Discover Devices**: Use device_list to access discovered devices
3. **Control Devices**: Call appropriate methods on device instances
4. **Monitor Status**: Use status retrieval methods to monitor device states
5. **Clean Up**: Call close() to properly shut down the API

### Example Workflow
```python
from hex_device import HexDeviceApi

# Initialize API
api = HexDeviceApi(ws_url="ws://192.168.1.100:8439", control_hz=500)

# Discover and control devices
try:
    while not api.is_api_exit():
        # Access devices
        for device in api.device_list:
            # Control logic here
            pass
finally:
    # Clean up
    api.close()
```