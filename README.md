# SSS2
The code base for the Teensy 3.6 based Smart Sensor Simulator 2. This SSS2 is primarily designed to simulate sensors for heavy vehicle electronic control units; however, it can be used for many other things. It is a multitool for vehicle systems. It can be used in a forensic context to simulate the presence of a vehicle and reduce the number of fault codes present when turning on the system again. 

## Software Design
There are three sets of files needed to make the SSS2 work. All the software in this repository is for Arduino.

  1. Board Definitions. This file declares the pin settings and defines the functions needed to communicate with the chips on board. For example, there is a difference between the SSS2-R02 boards and the SSS-R03 boards in the way the chip select pin is accessed for the digitial potentiometers. This file is `SSS2_board_defs_rev_X.h`
  2. Settings processor. This file sets up the function calls for all of the commands. This is decoupled from the communications module to enable different ways of communicating with the SSS2. This file is `SSS2_functions.h`
  3. Communications Interface. This the main wrapper file that provides an interface between the user and the SSS2 functions. The primary method of communications is Serial over USB, but CAN, LIN, J1708, WiFi (with the ESP8266) or others are enabled by customizing this main file. This file is usually called `SSS2_communications_over_serial.ino` and contains functions for all SSS2 variants. This means the lower level files need to tolerate differences.
  
### Board Definitions
The Schematics for the boards are available in the docs directory. The revision of the board is indicated in the serial number: SSS2-R0X, where X is the board revision. A define statment is at the top of the `SSS2_boards_defs.h` file to use in the programs. The following constants are defined: 

`SSS2_BOARD_REVISION` - an integer representing the board revision.

#### Pin Defintions
All pin defintions are `const int8_t` data types so they can take on a value of -1. If the pins are not used for a particular board revision, they should be set to -1.

`greenLEDpin` - output for the pin that drives the green led.

`redLEDpin` - output for the pin that drives the red led.

`CSCANPin` - output for the chip select pin for the Microchip MCP2515 Stand alone CAN transciever over SPI.

`PWMPins` - output pin array for pulse width modulated signal providing the input for an opamp that converts the 3.3V signal to a 5V signal.

`CSconfigAPin` - output pin for the chip select pin for the first Analog Devices ADG1414 SPI controlled CMOS switching chip.

`CSconfigBPin` - output pin for the chip select pin for the other Analog Devices ADG1414 SPI controlled CMOS switching chip.

`buttonPin` -  input pin that is pulled up for the push button input.

`encoderAPin` - input pin used for the quadrature knob.

`encoderBPin` - input pin used for the quadrature knob.

`IH1Pin` - output pin to drive a high current, high side switch from the Infineon BTM7710 driver chip.

`IH2Pin` - output pin to drive a high current, high side switch from the Infineon BTM7710 driver chip.

`IL1Pin` - output pin to drive a high current, low side switch from the Infineon BTM7710 driver chip.

`IL2Pin` - output pin to drive a high current, low side switch from the Infineon BTM7710 driver chip.

`ignitionCtlPin` - output pin to drive an N-Channel MOSFET that drives a relay solenoid to switch 12 V.

`analogInPins` - An input pin array for the Analog to Digital Converter.

This file defines a `setPinModes()` function to use when initializing the board. It sets all the pin modes and writes default values to them. This is what turns on the red LED at the beginning. 

#### Settings Variables
There are SPI connected potentiometers in the SSS that all have thier own settings. We define the number of these with `numSPIpots` which is usually 16. Then we can define arrays that holds all the potenetiometer settings. Since these potentiometers have settings for both the wiper position and the terminal connections, there are two arrays: `SPIpotWiperSettings` and `SPIpotTCONSettings`. These SPI potentiometers are labeled U1 through U16 in the schematics.

There are I2C connected potentiometers in the SSS2 Rev 3 and greater boards. These potentiometers use addressing to access the chip controls. The MCP45HV51 digital potentiometer can have up to 4 unique addresses on one I2C line. Three of these potentiometers are used for outward facing ports and the other is used to adjust the voltage on the high current regulator. The arrays that stores these settings are `I2CpotWiperSettings` and `I2CPotTCONSettings`. 

There are also settings for all the switch or boolean variables set up. The PWM values and DAC values are given as well.
 
### SSS2 Functions
The SSS2_functions file is a header file that defines all the functions needed to convert a String command into a setting output. There are 2 setting strings that are used: `commandPrefix` and `commandString`. The command prefix is used but the communications module to call a function. The `commandString` variable is parsed by the function to determine what it needs to do. 

### SSS2 Settings
The following are the enumerated settings for the SSS2. These settings are set using a lookup table, which is a long if-then-else structure. Teh settings are set when the command `setSetting` is called. The function is declared as: 

```int16_t setSetting(uint8_t settingNum, int settingValue, bool debugDisplay)```

 where `settingNum` is from the enumerated list below and the `settingValue` is the value for which to set the setting. The debugDisplay provides additional serial console ouptut from the executed command. The function returns the value of the setting as a confirmation it was set. The list of settings is as follows:
 
  1. Digital Potentiometer  1 Wiper, Port  1 (J24-1)
  2. Digital Potentiometer  2 Wiper, Port  2 (J24-2)
  3. Digital Potentiometer  3 Wiper, Port  3 (J24-3)
  4. Digital Potentiometer  4 Wiper, Port  4 (J24-4)
  5. Digital Potentiometer  5 Wiper, Port  5 (J24-5)
  6. Digital Potentiometer  6 Wiper, Port  6 (J24-6)
  7. Digital Potentiometer  7 Wiper, Port  7 (J24-7)
  8. Digital Potentiometer  8 Wiper, Port  8 (J24-8)
  9. Digital Potentiometer  9 Wiper, Port  9 (J24-9)
  10. Digital Potentiometer 10 Wiper, Port 10 (J24-10)
  11. Digital Potentiometer 11 Wiper, Port 11 (J24-11)
  12. Digital Potentiometer 12 Wiper, Port 12 (J24-12)
  13. Digital Potentiometer 13 Wiper, Port 13 (J18-11)
  14. Digital Potentiometer 14 Wiper, Port 14 (J18-12)
  15. Digital Potentiometer 15 Wiper, Port 15 (J24-15)
  16. Digital Potentiometer 16 Wiper, Port 16 (J24-16)
  17. Vout2-A, Port 18 (J18- 2)
  18. Vout2-B, Port 19 (J18- 3)
  19. Vout2-C, Port 20 (J18- 4)
  20. Vout2-D, Port 21 (J18- 5)
  21. Vout2-E, Port 22 (J18- 6)
  22. Vout2-F, Port 23 (J18- 7)
  23. Vout2-G, Port 24 (J18- 8)
  24. Vout2-H, Port 25 (J18- 9)
  25. U1 & U2 P0A, Ports  1 and 2
  26. U3 & U4 P0A, Ports  3 and 4
  27. U5 & U5 P0A, Ports  5 and 6
  28. U7 & U2 P0A, Ports  7 and 8
  29. U9 & U10 P0A, Ports  9 and 10
  30. U11 & U12 P0A, Ports 11 and 12
  31. U13 & U14 P0A, Ports 13 and 14
  32. U15 & U16 P0A, Ports 15 and 16
  33. PWM 1, Ports 13 (J24-13) and 31 (J18-15)
  34. PWM 2, Ports 14 (J24-14) and 32 (J18-16)
  35. PWM 3, Port 27 (J18-10)
  36. PWM 4, Port 17 (J18-1)
  37. Port 10 or 19, (J24-10)
  38. Port 15 or 18, (J24-15)
  39. CAN1 or J1708, (J24-17 & J24-18)
  40. Port 31 & 32 or CAN2, (J18-15 and J18-16)
  41. CAN0 Termination Resistor, R44
  42. CAN1 Termination Resistor, R45
  43. CAN2 Termination Resistor, R46
  44. LIN Master Pullup Resistor, R59
  45. 12V Out 1 (H-Bridge), Port 26 (J18-10)
  46. 12V Out 2 (H-Bridge), Port 11 (J24-11)
  47. Ground Out 1 (H-Bridge), Port 17 (J18- 1)
  48. Ground Out 2 (H-Bridge), Port 12 (J24-12)
  49. High Voltage Adjustable Output, (J24-19 and J18-11)
  50. Ignition Relay, (J24-20)
  51. Dig. Pot.  1 Terminal Connect, Port  1 (J24- 1)
  52. Dig. Pot.  2 Terminal Connect, Port  2 (J24- 2)
  53. Dig. Pot.  3 Terminal Connect, Port  3 (J24- 3)
  54. Dig. Pot.  4 Terminal Connect, Port  4 (J24- 4)
  55. Dig. Pot.  5 Terminal Connect, Port  5 (J24- 5)
  56. Dig. Pot.  6 Terminal Connect, Port  6 (J24- 6)
  57. Dig. Pot.  7 Terminal Connect, Port  7 (J24- 7)
  58. Dig. Pot.  8 Terminal Connect, Port  8 (J24- 8)
  59. Dig. Pot.  9 Terminal Connect, Port  9 (J24- 9)
  60. Dig. Pot. 10 Terminal Connect, Port 10 (J24-10)
  61. Dig. Pot. 11 Terminal Connect, Port 11 (J24-11)
  62. Dig. Pot. 12 Terminal Connect, Port 12 (J24-12)
  63. Dig. Pot. 13 Terminal Connect, Port 13 (J18-11)
  64. Dig. Pot. 14 Terminal Connect, Port 14 (J18-12)
  65. Dig. Pot. 15 Terminal Connect, Port 15 (J24-15)
  66. Dig. Pot. 16 Terminal Connect, Port 16 (J24-16)
  67. PWM1 Connect, Port 13 (J24-13)
  68. PWM2 Connect, Port 14 (J24-14)
  69. PWM3 Connect, Port 27 (J18-10)
  70. PWM4 Connect, Port 17 (J18- 1)
  71. LIN to Shield Connect, (J10- 5)
  72. LIN to Port 16 Connect, Port 16 (J24-16)
  73. U28 (U1-U8)  P0A Enable, (J24-1 to J24-8)
  74. U31 (U9-U16) P0A Enable, (J24-9 to J24-16)
  75. Digital Potentiometer 28 Wiper, Port 28 (J18-12)
  76. Digital Potentiometer 29 Wiper, Port 29 (J18-13)
  77. Digital Potentiometer 30 Wiper, Port 30 (J18-14)
  78. Dig. Pot. 28 Terminal Connect, Port 28 (J18-12)
  79. Dig. Pot. 29 Terminal Connect, Port 29 (J18-13)
  80. Dig. Pot. 30 Terminal Connect, Port 30 (J18-14)
  81. PWM1 Frequency, Port 13 (J24-13)
  82. PWM2 Frequency, Port 14 (J24-14)
  83. PWM3 Frequency, Port 27 (J18-10)
  84. PWM4 Frequency, Port 17 (J18-1)
  
Some setting take binary values of 0 or 1. For example, if the SSS2 recieves a serial command that says `50,1` it will interpret that to set seting number 50 to true, which means to close the ignition key switch relay. Similarly, the command `45,0` will turn off the 12V output on J18-10. 

  - Binary Settings
  - Terminal Settings
  - Digital Potentiometer Settings
  - Voltage Output Settings
  - PWM Frequency Settings
  - PWM Duty Cycle Settings


 