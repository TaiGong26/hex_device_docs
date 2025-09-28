**Note**: This document only records the **public** function interfaces of all classes. If you modify the `hex_device_python` library or call **non-public** functions, any resulting issues will not be supported.

- Device communication with lower-level systems and thread management are provided by the `HexDeviceApi` class, which serves as the unified interface for using HexDevice.
- The system provides two core base classes: `DeviceBase` and `MotorBase`. All device classes inherit from `DeviceBase`, while device classes with motors inherit from `MotorBase`, implementing common interfaces between devices.

# Common Interfaces
**[HexDeviceApi](API-Common)**  
**[DeviceBase](API-Common#devicebase)**  
**[MotorBase](API-Common#motorbase)**  

# Chassis
**[ChassisMaver](API-ChassisMaver)**  
**[ChassisMark2](API-ChassisMark2)**  

# Robotic Arm
**[ArmArcher](API-ArmArcher)**  

# Hands
**[Hands](API-Hands)**  