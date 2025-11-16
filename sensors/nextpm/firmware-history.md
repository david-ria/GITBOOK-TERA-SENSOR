---
description: >-
  NB: user guide and Datasheet will always be updated accordingly to the latest
  active firmware version
icon: square-terminal
---

# Firmware history



## May 2025 : 1047 Firmware update

* Modbus configuration of parity available
* Unique internal serial number for identification of each NextPM with a command

### Simplified protocol optimization:

* Possibility to activate or deactivate the heating system with simplified protocol
* Total Suspended Particles (TSP) available
* 5 channels available
* Improved temperature and relative humidity measurement&#x20;

### Modbus optimization:

* Sleep mode
* Calibration parameters implemented internally as y=ax+b function to give users the possibility to adjust PM concentration (only available for PM1, PM2.5 and PM10 in µg/m3)



***

## Sept 2024: 1046 Firmware update

* Improvement of the temperature and relative humidity measures with and without heating system
* Modification of the behaviour of the heating system, with automatic regulation shifting from 0% up to 100% if relative humidity exceeds 65%
* New modbus registers added to get the particle size distribution on the whole range of measurement of the NextPM ,from 0.3-10µm in number of particle per liter
  * 0.3 - 0.5 µm
  * 0.5 - 1 µm
  * 1 - 2.5 µm
  * 2.5 - 5 µm
  * 5 - 10 µm
* Total Suspended Particles (TSP) added to modbus register
