[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://paypal.me/bagges?country.x=DE&locale.x=de_DE)

# deye-esp32-bridge

This PCB and Software is used to create a board which communicates with your Deye/Sunsynk inverter and also the BMS of your battery (SeplosBMS and PaceBMS are supported for now).

<img src="img/board.svg">
<img src="img/schematic.svg">
<img src="img/3d.jpg">

# BOM
 - PCB (designed with easyeda and produced by jlcpcb)
 - [AZDelivery ESP32 Dev v4](https://amzn.to/41Sd0v4)
 - [2x RS485 Serial Converter](https://amzn.to/3L1OmBq)
 - [DC/DC Step Down Module](https://amzn.to/3oFFiur)
 - [3x RJ45 connectors](https://amzn.to/3AqmPo8)
 - [12V din rail power supply](https://amzn.to/3VNIEYH)

# Prepare RS485 Serial Converter
Make sure that your RS485 converter is terminated by about 120R. 
Measure the resistance between B- and A+. Should be something between 110-130 Ohms. 
If you cannot measure any resistance, you need to shorten the the two small pins next to the B- connector. 
On my boards it was labled with R13.

# Deye Setup
 - Goto Advanced Function and set to Slave and Modbus SN 01

# Flash ESP32
 - Adapt your wifi settings in the secrets.yaml
 - Connect your PC to the micro USB and run: ```esphome run deye-esp32-bridge.yaml```

# Wiring
 - Connect any 7-24V DC source to the screw headers (do not use CN2 PINS 7+8 from deye inverter, they only work for about 100mA). Make sure that JP2 is closed.
 - As an alternative you can connect directly to the USB of the ESP32. Make sure that JP2 is not closed. This method makes often problems with the RS485 connection.
 - Connect your inverters BMS RJ45 to the CAN IN RJ45 of the PCB.
 - If you want to connect your inverter and BMS via CAN. connect your BMS to the BMS IN RJ45 of the PCB.
 - Connect your Seplos BMS to RJ3
 - I used RJ45 cables without shield connected to the RJ45 connector

## Component Description
| Component | Function |
|-----------|--------------------------------------------------------|
| JP1       | If closed, connects GND from Deye to GND of the BMS    |
| JP2       | If closed, connects the DCDC Converter to the ESP32    |
| JP4       | If closed, connects GND from Deye to GND of the PCB    |
| JP5       | If closed, connects GND from the BMS to GND of the PCB |
| U2        | Additional general purpose I/O / GND / 5V / 3V3        |
| R1        | Optional resistor, needed if connecting DS18B20 to U4  |

 
# Software
 - The esphome configuration file for deye is based on the work of klatremis (https://github.com/klatremis/esphome-for-deye)
 - The esphome configuration file for seplos is based on the work of syssi (https://github.com/syssi/esphome-seplos-bms)
 
# Disclaimer

This repository contains files for demonstration purposes only. Use the files on your own risk. I am not responsible for any damage!

# License

Shield: [![CC BY-NC-SA 4.0][cc-by-nc-sa-shield]][cc-by-nc-sa]

This work is licensed under a
[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License][cc-by-nc-sa].

[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa]

[cc-by-nc-sa]: http://creativecommons.org/licenses/by-nc-sa/4.0/
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
[cc-by-nc-sa-shield]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg
