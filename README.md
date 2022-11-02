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
### Delta Modbus Spindle class
### EtherCAT Spindle class
