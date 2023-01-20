# deye-esp32-bridge

This PCB and Software is used to create a board which communicates with your Deye/Sunsynk inverter.

<img src="img/board.svg">
<img src="img/schematic.svg">
<img src="img/3d.jpg">

# Used Hardware
 - PCB (designed with easyeda and produced by jlcpcb)
 - AZDelivery ESP32 Dev v4
 - RS485 Serial Converter
 - DC/DC Step Down Module
 - 2x RJ45 connectors

# Flash ESP32
 - Adapt your wifi settings in the secrets.yaml
 - Connect your PC to the micro USB and run: ```esphome run deye-esp32-bridge.yaml```

# Wiring
 - Connect any 7-24V DC source to the screw headers (CN2 PINS 7+8 from deye inverter will work). Make sure that JP2 is closed.
 - As an alternative you can connect directly to the USB of the ESP32. Make sure that JP2 is not closed.
 - Connect your inverters BMS RJ45 to the IN RJ45 of the PCB.
 - Connect your BMS to the BMS IN RJ45 of the PCB.
 
# Software
The esphome configuration file is based on the work of klatremis (https://github.com/klatremis/esphome-for-deye)

# Disclaimer

This repository contains files for demonstration purposes only. Use the files on your own risk. I am not responsible for any damage!


