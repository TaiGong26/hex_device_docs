# Unrelease
1. Lift support


# Version Log

## Version Naming Convention: A.B.C

- **A (Major Version)**: Different versions indicate significant architectural changes that cannot be adapted through software/hardware upgrades
- **B (Minor Version)**: Different versions require hardware upgrades before they can be used
- **C (Patch Version)**: Different versions indicate software-only upgrades that can be directly upgraded and used

## History version

## 1.1.2
Add support for HandsHtGp100.

## 1.1.3
HandsHtGp100 can use mit mode now.

## 1.1.4
ChassisMaver and ChassisMark2 were merge to the same class - chassis.

## 1.1.6
Chassis add start & stop func, it means that the chassis must explicitly call the start() function to respond to commands, and it can safely exit control by calling the stop() function.

## 1.1.7
Rename arm class and test ArmSaber finished.

## 1.2.1
- Added session management - all current connections are assigned a session ID, only the first session to send api_init command can become the session holder
- Added mode switching for robotic arm - now supports xview for mode switching
- Robotic arm commands are divided into two types: ArmExclusiveCommand and ArmSharedCommand. Control commands are ArmExclusiveCommand and only respond to control commands from the current session holder
- **For chassis:** Users currently using software packages version 1.1.6 or higher can directly upgrade to this version.

## 1.3.1
- Added protocol version validation; the API now logs errors when the connected firmware version is unsupported.
- Optional devices have been moved to the `SecondaryDevice` flow, each assigned a device ID so multiple devices of the same type can be distinguished.
- Added KCP transport support, providing lower communication latency and faster recovery from packet loss.

## 1.3.2
- Add support for the Saber 7-DOF arm, but note that MIT mode is not enabled by default.
- Fix incorrect references to chassis variables.
- Add function to retrieve the raw encoder values for the arm.
- Add function to check timeout for chassis.

## 1.3.3
- Add is_timeout() for chassis.
- Add get_encoders_to_zero() for arm.
- Add torque control limit to saber.
- Add linear lift support.

## 1.3.4
- Use deque to buff motor data.
- Fix logger handler error.

## 1.3.5
- Remove clear_new_data_flag(), has_new_data() checks for new data based on the queue length now.
- Change the name of arm type.
- Implemented PTP time synchronization.
- Improve the program's exit process.

## 1.3.6 (2025.12.11)
Fix:
- Fixed the issue of unexpected reduction in the data queue caused by incorrect internal variable references.
- Fixed the issue where the control frequency was not passed down properly.

Feat:
- IPv6 connectivity is now supported.

## 1.3.7 (2025.12.12)
Fix:
- Fix IPv4 interface crash.

## 1.3.8 (2025.12.12)
Fix:
- Sync chassis simple data deque & motor data deque.
Feat:
- Add imu & gamepad support.

## 1.3.9 (2025.12.17)
Feat:
- Now the ReportFrequency will follow the control hz from api init.

## 1.3.10 (2025.12.23)
Feat:
- Add support for RtArmArcherY6_H1.

## 1.3.11 (2025.12.23)
Fix:
- Add joint limits params for RtArmArcherY6_H1.

## 1.3.12 (2025.12.24)
Fix:
- Ipv6 address parse error on kcp.
- Argparse lib raise when use python3.14

## 1.3.13 (2025.12.27)
Fix:
- Hands's position calc error.

## 1.3.14 (2026.1.9)
Fix:
- Fix missing configuration for firefly_y6_h1 config.

## 1.3.15 (2026.1.23)
Refactor:
- Rename RtLotaP1 to RtIotaP1.

Feat:  
- Add support for archer_y6_h1.
- Add support for SdtHandGp80G1.
- Add support for ZetaLift.
- Add support for Hello.
- Add proto version & Add hex_device version log.

## 1.3.16 (2026.2.2)
Feat:
- The set_rgb_stripe_command function no longer raises an error when the number of parameters does not match. Instead, it automatically fills in missing parameters with zeros and imposes range restrictions on the input parameters. 
- The MotorBase class has been enhanced with the method construct_mit_command.

Refactor:
- SdtHello has been updated to inherit from the MotorBase class to unify the interface (functionality not implemented).

## 1.3.17 (2026.2.3)
Feat:
- Currently, the logs will print the IP address and port number of the API's current connection target.
- Add parsing for the warning field in the MotorStatus structure.

Refactor:
- Change to directly report the error code when there is a motor error, instead of just indicating that an error has occurred.

## 1.3.18 (2026.2.15)
Refactor:
- Change joint limits for archer_y6、hello_y6、firefly_y6、hello_firefly_y6.
- Invalid websocket url will exit directly now.

## 1.3.19 (2026.3.25)
**For Arm:**
Fix:
- Fix the incorrect setting of timeout check status.
- Fix Position mode check escape vulnerability.

## 1.3.20 (2026.3.25)
**For Lift**
Fix:
- Fix log_err when emergency stop is None.
**For chassis**
Refactor:
- Change log level when is not the session holder.

## 1.4.0 (2026.3.1)
**For Arm**
Feat:
- Some special control mode is support.
**For MotorBase**
Refactor:
- warning!!! The unit of mit's position is change to rad.
- This version not support 1.3 hardware version.

## 1.4.1 (2026.3.16)
**For Api**
Feat:
- Add stream mode support with callback for data handling.
- Add export for proto message types.
**For Arm**
Feat:
- Add func is_timeout.
- Merge 1.3.19 version commit.
**For Hands**
Feat:
- Add Gr100 support.
