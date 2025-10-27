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