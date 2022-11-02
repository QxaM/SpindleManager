# SpindleManager
This library handles spindle control in CODESYS or CODESYS based softwares. Vast majority of spindle applications are usually rather simple start/stop and velocity control. Logic of this control is similar for different control protocols or communication mechanism. In this library such unified logic is achieved with spindle state machine as seen below:
![obraz](https://user-images.githubusercontent.com/109360131/199506887-51aa946c-bda3-47a8-9ff8-1214ad516a49.png)

## Abstract spindle manager class
To achieve this logic abstract spindle manager class was created. Class implements spindle logic as above, moreover by default provides user with basic necessary inputs outputs. 

### Inputs:
- **i_runCW**, **i_runCCW** - allow for running spindle in either direction
- **i_reset** - resets spindle if any error occurs
- **i_targetVelocity** - rotation velocity target for spindle command in RPM

### Outputs:
- **q_actualVelocity** - output with feedback of actual velocity. Since feedbacks will differ between different control methods this is left without implementation. Implement this output when inheriting this block
- **q_error** - outputs when drive is in error
- **q_spindleStopped** - outputs when spindle is stopped
- **q_spindleTargetReached** - outputs when spindle rotating speed command is reached by actual velocity

### State machine implementation
State machine is implemented in main body of the class. Additionally, update method was created to check and update ramps and check and switch to error state if error occurs.

![obraz](https://user-images.githubusercontent.com/109360131/199514850-c8f1019e-2028-40c7-9c18-d2edeefc7484.png)

## Interfaces
In the library three interfaces were create to represent key elements in drive control. Abstract class implements this interfaces. User have to create implementation of this interfaces if abstract class is inherited. In classes provided in this library such implementation was created.

Those interfaces are usually present in most drives and represent - error handling interface, ramps handling interface (ie. sending ramps to drive), motion interface, that handles most motion related actions - powering drive or running drive with set velocity:

![obraz](https://user-images.githubusercontent.com/109360131/199517050-c2245a9a-d2fb-400e-8386-69ba32e994ec.png)

## Library defined drive control classes
This library defines couple drive control classes/function blocks. They are 3 most common use cases of spindles - analog spindle, modbus controlled spindle for Delta Electronics drive (either ASCII, RTU or TCP), CiA402 standard controlled spindle for drives supporting CiA402 standard protocol. Such drive in the library and later in this manual is refered as EtherCAT controlled drive, but EtherCAT could be switched for CANOpen supporting DS402/CiA402 standard.

### Analog Spindle class
Controlling drives via analog singal is usually the simplest and cheapest way, since most of VFD drives have analog signal built in by default. Since this is the simplest way to control a drive, we need several additional signals as feedback from a drive or outputs to drive. Those signals was furhter explained below. Class inherits abstract spindle class with all its logic and implements interfaces explained above. Class did not implement ramps interface, since no ramps can be changed without using some kind of communication. Ramps have to be set as constants in drives parameters.

### Inputs:
- i_driveInError - feedback from a drive, that is in error
- i_drivePowerON - feedback from a drive, that drive's power stage is powered ON
- i_driveStopped - feedback from a drive, that drive is stopped (output frequency = 0)
- i_driveTargetReached - feedback from a drive, that drive has reached it's command frequency
- i_driveActualVelocity - feedback from a drive (ie. analog output from drive) with drive's actual rotating frequency

All of the above funcionality coudl be omitted with proper constant settings of those inputs. For example, if we don't need to use i_driveTargetReached function, we could set it to constant TRUE. Class won't check target reached, but whole functionality of drive will be intact. Similarly, Error coudl be empty or false - then we will not check errors on drive, power ON coulb be TRUE, etc.

### Outputs
- q_driveReset - output used for reseting drive errors
- q_driveEnable - output used for enabling power stage of a drive
- q_driveAnalogCommand - analog speed command of a drive

### Delta Modbus Spindle class
Delta Electronics VFDs have Modbus ASCII/RTU protocol build in by default. Modbus communication in exactly the same among different VFD series, so class could be used with any Delta VFD. Library author believes this is the best cheap way to control Delta VFDs - since modbus is a standard, no additional communication cards are needed and with communication we could minimize number of electric connections. Nevertheless, several additional signals are necessery to map to the class and are explained below. Class inherits abstract spindle class with all its logic and implements interfaces explained above.

### Inputs
- i_warnErrorCode - feedback from a drive with warn or error codes. In Delta this maps to **2100h** register
- i_statusWord - feedback from a drive with status of a drive. In Delta this maps to **2101h** register
- i_outputFrequency - feedback from a drive with its' actual output frequency. In Delta this maps to **2102h** register.
- i_accRamp, i_dccRamp - acceleration/deceleration ramp times in seconds, in floating point format (REAL), entered by a user. Can be empty (then default is 0.5s), but cannot be 0

### Outputs
- q_controlWord - control word sent to a drive. In Delta this maps to **2000h**
- q_frequencyCommand - frequency command sent to a drive. In Delta this maps to **2001h**
- q_driveReset - drive reset command. In Delta this maps to **2002h** **bit1**.
- q_accRamp, q_dccRamp - acceleration/deceleration ramp times sent to a drive via Modbus communication. This maps to parameters 01-12 = **010Ch** and 01-13 = **010Dh** respectively in Delta VFDs.

### EtherCAT Spindle class
Class that allows to control drives via EtherCAT communication (or any other communication supporting CiA402 protocol). A drive, to be controlled by this block, have to be set to velocity control via register **6060h** - Mode of Operation, by setting **2**. Delta VFDs are by default set to velocity mode after power cycle, so no additional settings are needed. Class extends abstract spindle class with all its logic and implements interfaces explained above, but several additional inputs and outputs were neceserry:

### In/outs:
- i_drive - EtherCAT slave IEC object, created by CODESYS for every EtherCAT slave.

### Inputs:
- i_statuWord - status word feedback from a drive. Mapped to EtherCAT status word register - **6041h**
- i_actualVelocity - feedback from a drive with its' actual rotating velocity. Mapped to EtherCAT actual velocity register (in Velocity mode) - **606Ch**
- i_accTime, i_dccTime - acceleration/deceleration ramp times in seconds, in floating point format (REAL), entered by a user. Can be empty (then default is 0.5s), but cannot be 0

### Outputs
- q_controlWord - control wrod sent to a drive. Mapped to EtherCAT control word register - **6040h**
- q_targetVelocity - velocity command sent to a drive. Mappet to EtherCAT target velocity register - **60FFh**

## Getting started
To get started using this library implement it in codesys via Tools->Library Repository->Install

![obraz](https://user-images.githubusercontent.com/109360131/199534911-55964d2d-69b8-427d-a046-002ff551893e.png)

![obraz](https://user-images.githubusercontent.com/109360131/199535158-d722abda-f623-462c-8ed5-06410665930e.png)

After that, declare, call and map in/outs of one of included spindle manager classes. Extend abstract spindle manager class if you want to create specific spdinle manager class. Extend one of provided classes to match your specific needs.

## Drive parameters
Remember that to use this blocks you have to set specific parameters in drive. Depending on drive and control method it will be:
- control method - external terminals, modbus communication, etherCAT communication
- frequency control method - analog, modbus communication, etherCAt communication
- digital input and output functions - depanding on needs and drive
- analog input and output functions - as above
- modbus communication parameters - station adress, communication format and speed, error handling 
