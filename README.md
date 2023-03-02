- [Library Information](#org96ddff7)
- [Stepper Motors](#orgc05b014)
- [Stepper Motor Controllers and Drivers](#org22f0195)
- [Communication](#org82dacdf)
- [Defaults](#org4a7b4a1)
- [Examples](#orga2d300a)
- [Hardware Documentation](#org51c0040)

    <!-- This file is generated automatically from metadata -->
    <!-- File edits may be overwritten! -->


<a id="org96ddff7"></a>

# Library Information

-   **Name:** TMC2209
-   **Version:** 8.0.7
-   **License:** BSD
-   **URL:** <https://github.com/janelia-arduino/TMC2209>
-   **Author:** Peter Polidoro
-   **Email:** peter@polidoro.io


## Description

The TMC2209 is an ultra-silent motor driver IC for two phase stepper motors with both UART serial and step and direction interfaces.

![img](./images/TMC2209.png)


<a id="orgc05b014"></a>

# Stepper Motors

From Wikipedia, the free encyclopedia:

A stepper motor, also known as step motor or stepping motor, is a brushless DC electric motor that divides a full rotation into a number of equal steps. The motor's position can be commanded to move and hold at one of these steps without any position sensor for feedback (an open-loop controller), as long as the motor is correctly sized to the application in respect to torque and speed.

[Wikipedia - Stepper Motor](https://en.wikipedia.org/wiki/Stepper_motor)


<a id="org22f0195"></a>

# Stepper Motor Controllers and Drivers

Stepper motors need both a controller and a driver. These may be combined into a single component or separated into multiple components that communicate with each other, as is the case with the TMC2209 stepper motor driver. One controller may be connected to more than one driver for coordinated multi-axis motion control.


## Stepper Motor Controller

A stepper motor controller is responsible for the commanding either the motor kinetics, the torque, or the motor kinematics, the position, speed, and acceleration of one or more stepper motors.


## Stepper Motor Driver

A stepper motor driver is responsible for commanding the electrical current through the motor coils as it changes with time to meet the requirements of the stepper motor controller.


## TMC2209 Stepper Motor Driver and Controller Combination

The TMC2209 is a stepper motor driver and it needs a stepper motor controller communicating with it.

The TMC2209 can be used as both a stepper motor driver and stepper motor controller combined, independent from a separate stepper motor controller, but it is limited to simple velocity control mode only, with no direct position or acceleration control.

In velocity control mode, the velocity of the motor is set by the method moveAtVelocity(int32\_t microsteps\_per\_period). The actual magnitude of the velocity depends on the TMC2209 clock frequency. The TMC2209 clock frequency (fclk) is normally 12 MHz if the internal clock is used, but can be between 4 and 16 MHz if an external clock is used.

In general: microsteps\_per\_second = microsteps\_per\_period \* (fclk Hz / 2^24)

Using internal 12 MHz clock (default): microsteps\_per\_second = microsteps\_per\_period \* 0.715 Hz

Crude position control can be performed in this simple velocity control mode by commanding the driver to move the motor at a velocity, then after a given amount of time commanding it to stop, but small delays in the system will cause position errors. Plus without acceleration control, the stepper motor may also slip when it attempts to jump to a new velocity value causing more position errors. For some applications, these position errors may not matter, making simple velocity control good enough to save the trouble and expense of adding a separate stepper controller.

Most of this library's examples use the simple velocity control mode to test the driver independently from a separate stepper motor controller, however in most real world applications a separate motor controller is needed, along with the TMC2209 and this library, for position and acceleration control.


### Microcontroller Stepper Motor Controller

One controller option is to use just a single microcontroller, communicating with the TMC2209 over both the UART serial interface and the step and direction interface.

![img](./images/microcontroller_controller_driver.png)


### TMC429 and Microcontroller Stepper Motor Controller

Another controller option is to use both a microcontroller and a separate step and direction controller, such as the TMC429.

![img](./images/TMC429_controller_driver.png)


<a id="org82dacdf"></a>

# Communication

The TMC2209 driver has two interfaces to communicate with a stepper motor controller, a UART serial interface and a step and direction interface.

The UART serial interface may be used for tuning and control options, for diagnostics, and for simple velocity commands.

The step and direction interface may be used for real-time position, velocity, and acceleration commands. The step and direction signals may be synchronized with the step and direction signals to other TMC2209 chips for coordinated multi-axis motion.


## UART Serial Interface

[Wikipedia - UART](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter)

The TMC2209 communicates over a UART serial port using a single wire interface, allowing bi-directional operation for full control and diagnostics. It can be driven by any standard microcontroller UART or even by bit banging in software.

The UART single wire interface allows control of the TMC2209 with any set of microcontroller UART serial TX and RX pins. The single serial signal is connected to both the TX pin and the RX pin, with a 1k resistor between the TX pin and the RX pin to separate them.

The microcontroller serial port must be specified during the TMC2209 setup.

For example:

```cpp

#include <Arduino.h>
#include <TMC2209.h>

// Instantiate TMC2209
TMC2209 stepper_driver;
HardwareSerial & serial_stream = Serial1;

void setup()
{
  stepper_driver.setup(serial_stream);
}

```

![img](./images/TMC2209_serial.png)


### Arduino Serial

[Arduino Serial Web Page](https://www.arduino.cc/reference/en/language/functions/communication/serial)

On some Arduino boards (e.g. Uno, Nano, Mini, and Mega) pins 0 and 1 are used for communication with the computer on the serial port named "Serial". Pins 0 and 1 cannot be used on these boards to communicate with the TMC2209. Connecting anything to these pins can interfere with that communication, including causing failed uploads to the board.

Arduino boards with additional serial ports, such as "Serial1" and "Serial2", can use those ports to communicate with the TMC2209.


### Teeny Serial

[Teensy Serial Web Page](https://www.pjrc.com/teensy/td_uart.html)

The Teensy boards have 1 to 8 hardware serial ports (Serial1 - Serial8), which may be used to connect to serial devices.

Unlike Arduino boards, the Teensy USB serial interface is not connected to pins 0 and 1, allowing pins 0 and 1 to be used to communicate with a TMC2209 using "Serial1".


### Serial Baud Rate

The serial baud rate is the speed of communication in bits per second of the UART serial port connected to the TMC2209.

In theory, baud rates from 9600 Baud to 500000 Baud or even higher (when using an external clock) may be used. No baud rate configuration on the chip is required, as the TMC2209 automatically adapts to the baud rate. In practice, it was found that the baud rate may range from 19200 to 500000 without errors.

The higher the baud rate the better, but microcontrollers have various UART serial abilities and limitations which affects the maximum baud allowed. The baud rate may be specified when setting up the stepper driver.

1.  Arduino

    The maximum serial baud rate on typical Arduino boards is 115200, so that is the default, but other values as low as 19200 may be used.
    
    [Arduino Serial Baud Rate Web Page](https://www.arduino.cc/en/Reference/SoftwareSerialBegin)

2.  Teensy

    Teensy UART baud rates can go higher than many typical Arduino boards, so 500k is a good setting to use, but other values as low as 19200 may be used.
    
    [Teensy Serial Baud Rate Web Page](https://www.pjrc.com/teensy/td_uart.html)
    
    ```cpp
    
    #include <Arduino.h>
    #include <TMC2209.h>
    
    // Instantiate TMC2209
    TMC2209 stepper_driver;
    HardwareSerial & serial_stream = Serial1;
    const long SERIAL1_BAUD_RATE = 500000;
    
    void setup()
    {
      stepper_driver.setup(Serial1,SERIAL1_BAUD_RATE);
    }
    
    ```


### Serial Addresses

More than one TMC2209 may be connected to a single serial port, if each TMC2209 is assigned a unique serial address. The default serial address is "SERIAL\_ADDRESS\_0". The serial address may be changed from "SERIAL\_ADDRESS\_0" using the TMC2209 hardware input pins MS1 and MS2, to "SERIAL\_ADDRESS\_1", "SERIAL\_ADDRESS\_2", or "SERIAL\_ADDRESS\_3".

The TMC2209 serial address must be specified during the TMC2209 setup, if it is not equal to the default of "SERIAL\_ADDRESS\_0".

For example:

```cpp

#include <Arduino.h>
#include <TMC2209.h>

// Instantiate the two TMC2209
TMC2209 stepper_driver_0;
const TMC2209::SerialAddress SERIAL_ADDRESS_0 = TMC2209::SERIAL_ADDRESS_0;
TMC2209 stepper_driver_1;
const TMC2209::SerialAddress SERIAL_ADDRESS_1 = TMC2209::SERIAL_ADDRESS_1;
const long SERIALX_BAUD_RATE = 115200;

void setup()
{
  // TMC2209::SERIAL_ADDRESS_0 is used by default if not specified
  stepper_driver_0.setup(Serial1,SERIALX_BAUD_RATE,SERIAL_ADDRESS_0);
  stepper_driver_1.setup(Serial1,SERIALX_BAUD_RATE,SERIAL_ADDRESS_1);
}

```

![img](./images/TMC2209_serial_address.png)


## Step and Direction Interface


### Microcontroller Stepper Motor Controller

The step and direction signals may be output from a microcontroller, using one output pin for the step signal and another output pin for the direction signal.


### TMC429 and Microcontroller Stepper Motor Controller

The step and direction signals may be output from a dedicated step and direction controller, such as the TMC429.

A library such as the Arduino TMC429 library may be used to control the step and direction output signals.

[Arduino TMC429 Library](https://github.com/janelia-arduino/TMC429)


<a id="org4a7b4a1"></a>

# Defaults


## Automatic Current Scaling

Automatic current scaling is disabled by default, so a potentiometer connected to VREF will not set the current limit of the driver. Current settings are controlled by UART commands instead.

Use enableAutomaticCurrentScaling() to allow VREF to set the current limit of the driver.


## Automatic Gradient Adaptation

Automatic gradient adaptation is disabled by default.

Use enableAutomaticGradientAdaptation() to reenable.


<a id="orga2d300a"></a>

# Examples


## Wiring


### Teensy 4.0

![img](./images/TMC2209_teensy40.svg)


### Mega 2560

![img](./images/TMC2209_mega2560.svg)


### Wiring Documentation Source

<https://github.com/janelia-kicad/trinamic_wiring>


<a id="org51c0040"></a>

# Hardware Documentation


## Datasheets

[Datasheets](./datasheet)


## TMC2209 Stepper Driver Integrated Circuit

[Trinamic TMC2209-LA Web Page](https://www.trinamic.com/products/integrated-circuits/details/tmc2209-la)


## TMC429 Stepper Controller Integrated Circuit

[Trinamic TMC429 Web Page](https://www.trinamic.com/products/integrated-circuits/details/tmc429/)


## SilentStepStick Stepper Driver Board

[Trinamic TMC2209 SilentStepStick Web Page](https://www.trinamic.com/support/eval-kits/details/silentstepstick)


## BIGTREETECH TMC2209 V1.2 UART Stepper Motor Driver

[BIGTREETECH TMC2209 Web Page](https://www.biqu.equipment/products/bigtreetech-tmc2209-stepper-motor-driver-for-3d-printer-board-vs-tmc2208)


## Janelia Stepper Driver

[Janelia Stepper Driver Web Page](https://github.com/janelia-kicad/stepper_driver)