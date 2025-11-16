---
icon: lightbulb-exclamation-on
---

# NextPM User Guide

## 1. About NextPM <a href="#about-nextpm" id="about-nextpm"></a>

### 1.1 Generals <a href="#generals" id="generals"></a>

The NextPM sensor is a particulate matter measurement device which is able to detect PM1, PM2.5 and PM10 in mass concentration (µg/m3) and quantity (Nb/mL). All those data are available within 3 different average periods (10sec, 1 min and 15 min) to best suit the dynamic of the external environment.

It is also possible to measure particles classified in 5 different channels (1)(2):

* 0.3µm to 0.5µm
* 0.5µm to 1µm
* 1µm to 2.5µm
* 2.5µm to 5µm
* 5µm to 10µm

Sensor specifications, such as range, precision, conditions of use, etc.., are defined into the datasheet of the NextPM sensor which can be found at [https://tera-sensor.com/](https://tera-sensor.com/)

The “NextPM Innovative Technology” document highlights the performances of the sensor NextPM

The user can find explanations about fluidic and mechanics to be helped for its NextPM use/integration by referring to the NextPM [integration advice](broken-reference) chapter.

The user can find [accessories](broken-reference) either to quickly use and test the NextPM sensor but also to make an easier integration.



1. Only available since 0x1045 firmware version
2. Available since firmware 0x1047 for the 3 running averages (10 sec, 1 minute and 15 minutes)

***



### 1.2 Pinout and electrical considerations <a href="#pinout-and-electrical-considerations" id="pinout-and-electrical-considerations"></a>

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 103815.png" alt=""><figcaption></figcaption></figure>

| Pin # | Signal name |
| :---: | :---------: |
|   1   |     GND     |
|   2   |    DE/RE    |
|   3   |  Rx (input) |
|   4   | Tx (output) |
|   5   |     +5V     |
|   6   |     GND     |

* The NextPM must be power supplied with + 5 VDC / 0.2V.
* The average current consumption is lower than 70mA but it can go up to 210mA in case of heater activation (manual enable or automatic due to high relative humidity (see [Heater](broken-reference)). The average current consumption can be reduced to 20mA by turning the sensor into [sleep mode](broken-reference).
* The Tx and Rx signal logical level is TTL (+3.3V) and the Rx signal is +5V tolerant.
* The signal DE/RE is useful only in case of RS485 half duplex mode use, to drive the external RS485 transceiver, otherwise, it could be left unconnected (high impedance).
* The ADAM TECH _125CH-B-06_ or MOLEX _51021-0600_ connector can be purchased to connect the NextPM Sensor..\
  To connect the NextPM to a PC, a FTDI cable such as _TTL-232R-3V3 (supplier: FTDI chip)_ can be used.

<mark style="color:red;">Warning : do not connect NextPM sensor directly on the RS232 port of a PC or to a RS485 network.</mark>



### 1.3. Heater <a href="#heater" id="heater"></a>

The NextPM has the ability to automatically trigger and regulate its internal heater in case of high relative humidity. This provides a better measurement accuracy in those specific environmental conditions by drying the input air and the particles.

The heater is enabled from 60 %RH threshold and the heat generated is dependent on the measured relative humidity and so, the NextPM current consumption also (the additional current due to the heater can reach 140mA).

The user, for specific purposes, can enable/disable the automatic heater regulation but also set a specific value (see [Writing commands list](https://docs.google.com/document/d/1mDMpD21G-W-E4irnLSpVwDwx0-xLc66_a1_b9S5H0ZM/edit#heading=h.c4r5wogk8vuz) and [Writable registers list](broken-reference)).



### 1.4 Sleep mode <a href="#sleep-mode" id="sleep-mode"></a>

To save the system energy by reducing the average NextPM current consumption, the user can put the NextPM sensor into sleep mode by sending the corresponding command (see [Writing commands list](https://docs.google.com/document/d/1mDMpD21G-W-E4irnLSpVwDwx0-xLc66_a1_b9S5H0ZM/edit#heading=h.c4r5wogk8vuz) and [Writable registers list](broken-reference)). This reduces the current consumption below 20mA.

In this mode, the NextPM cannot provide any measurement as it has stopped the majority of its functionalities (fan, laser, heater) but its communication remains functional.

As at the power on of the sensor, at the sleep mode exit, the measurements are reliable after 15 seconds.

To save more energy, if the user has the capability to, it is better to produce ON/OFF cycles by managing the sensor power supply . This way, the consumption will be 0mA during the NextPM off phase and the average consumption will be considerably reduced.



### 1.5 Temperature and relative humidity measurement <a href="#temperature-and-relative-humidity-measurement" id="temperature-and-relative-humidity-measurement"></a>

The NextPM measures its internal temperature and relative humidity for its own need and these values are accessible thanks to reading commands (see [0x14 command reply](broken-reference) and [Readable registers list](broken-reference)).

However, these internal values are biased by internal heat of the sensor due to PCB power dissipation and heater activation to improve PM measurements in a humid environment. So, the internal temperature and relative humidity are only technical data.

Environmental external data are calculated and available with the Modbus protocol if needed, but not implemented yet with the simple com protocol.

It will not take into account the global casing used by the integrator and adjustments may be needed to fit with outdoor parameters.



### 1.6 Status code description <a href="#status-code-description" id="status-code-description"></a>

The NextPM makes continuous internal diagnosis in order to let the user know the state of the sensor and so, the validity of the measurements.

The state is coded on 2 bytes (1) and it contents flags correspondings to errors which can occur:

| Bit 15 | Bit 14 | Bit 13 | Bit 12 | Bit 11 | Bit 10 | Bit 9 | Bit 8         |
| ------ | ------ | ------ | ------ | ------ | ------ | ----- | ------------- |
| -      | -      | -      | -      | -      | -      | -     | Default State |

| Bit 7       | Bit 6        | Bit 5     | Bit 4      | Bit 3      | Bit 2     | Bit 1          | Bit 0       |
| ----------- | ------------ | --------- | ---------- | ---------- | --------- | -------------- | ----------- |
| Laser Error | Memory Error | Fan Error | T/RH Error | Heat Error | Not Ready | Degraded State | Sleep State |

* bit 0: set if the user has sent a sleep mode command or if the sensor is in default state (bit 8).\
  In simple protocol, the only authorized command still possible is a NextPM state read.
* bit 1: set each time a minor error is detected and confirmed. The corresponding minor error flag is set. In this state, the NextPM can still measure data but with less accuracy.\
  The minor errors are the following:
  * bit 3: Heat Error: the relative humidity stay above 60% during more than 10 minutes
  * bit 4: T/RH Error: the sensor reading are out of specification
  * bit 5: Fan Error: the fan speed is out of range but the fan is still working
  * bit 6: Memory Error: the sensor can’t access its memory, some internal smart functions will not be available.
  * bit 7: Laser Error: the photodetector is not detecting any particle for at least 240 secs resulting from a possible laser error.
* bit 2: set at power on, meaning that the sensor is starting and that measurements are not yet reliable until the bit is clear. It lasts 15 seconds.\
  This bit is also set after a wake-up of the sensor (sleep mode exit)
* bit 8: set if the fan has stopped working, after 3 restart attempts, the sensor will switch to the default mode, set the default state flag meanwhile the degraded state flag is clear and the sensor goes into sleep mode.



(1) The 8 most significant bits are only available using modbus protocol



***



## 2. NextPM communication <a href="#nextpm-communication" id="nextpm-communication"></a>

The NextPM sensor is able to either communicate using a [simple protocol](broken-reference) based only on proprietary UART frames or using [Modbus standard protocol](broken-reference).

For both communication protocols, the physical layer is TTL 3.3V logical levels.



### 2.1 Serial communication configuration <a href="#serial-communication-configuration" id="serial-communication-configuration"></a>

The serial communication must be configured as followed :

* Default baud rate : 115 200
* Number of data bits: 8
* Parity : even
* Number of stop bits: 1

Notes:

* The NextPM considers a timeout when there is 50ms of idle state between 2 bytes.
* The NextPM delay is 50 ms (beginning of the reply to a request).



### 2.2 Simple protocol <a href="#simple-protocol" id="simple-protocol"></a>

The simple protocol uses the following frame format:

* Reading:
  * Request: address (0x81) + command function code + checksum
  * Response: address (0x81) + command function code + data + checksum
* Writing:
  * Request: address (0x81) + command function code + data + checksum
  * Response: address (0x81) + command function code + checksum

The checksum is calculated in such a way that the sum of all the frame bytes is equal to a multiple of 256 (0x100): checksum = 0x100 – MOD ((sum of the other bytes), 256).



Examples :

```
0x81 + 0x16 + 0x69 = 0x100
```

```
0x81 + 0x21 + 0x55 + 0x09 = 0x100
```

```
0x81 + 0x12 + 0x00 + 0x00 + 0x0D + 0x00 + 0x0E + 0x00 + 0x0F+ 0x00 + 0x6A + 0x00 + 0x72 + 0x00 + 0x85 + 0xE2 = 0x300
```



#### 2.2.1 Reading commands list <a href="#reading-commands-list" id="reading-commands-list"></a>

The NextPM information’s reading can be realized through the following command frames:

| Cmd code | Description                                                                               | Command        |
| -------- | ----------------------------------------------------------------------------------------- | -------------- |
| 0x01     | Unique identifi cation number (Id + Batch Number)                                         | 0x81 0x01 0x7E |
| 0x11     | Concentrations (µg/m3 and Nb/mL) averaged over 10 seconds and updated every 1 second      | 0x81 0x11 0x6E |
| 0x12     | Concentrations (µg/m3 and Nb/mL) averaged over 1 min and updated every 10 seconds         | 0x81 0x12 0x6D |
| 0x13     | Concentrations (µg/m3 and Nb/mL) averaged over 15 min and updated every 60 seconds        | 0x81 0x13 0x6C |
| 0x14     | Temperature (°C) and humidity (%RH)                                                       | 0x81 0x14 0x6B |
| 0x16     | Sensor state                                                                              | 0x81 0x16 0x69 |
| 0x17     | Firmware version                                                                          | 0x81 0x17 0x68 |
| 0x22     | Modbus address                                                                            | 0x81 0x22 0x5D |
| 0x25     | Concentration in Nb/L on the 5 bins\* averaged over 10s and updated every 1 second        | 0x81 0x25 0x5A |
| 0x26     | Concentration in Nb/L on the 5 bins\* averaged over 1 minute and updated every 10 seconds | 0x81 0x26 0x59 |
| 0x27     | Concentration in Nb/L on the 5 bins\* averaged over 15 minutes and updated every 1 minute | 0x81 0x27 0x58 |



#### 2.2.2  Reading replies <a href="#reading-replies" id="reading-replies"></a>

* **2.2.2.1           0x11, 0x12, 0x13 replies**

<table><thead><tr><th>Address</th><th>Cmd code</th><th>State (1 byte)</th><th>PM1 Nb/mL (2 bytes)</th><th>PM2.5 Nb/mL (2 bytes)</th><th>PM10 Nb/mL (2 bytes)</th><th>PM1 µg/m3 (2 bytes)</th><th>PM2.5 µg/m3 (2 bytes)</th><th>PM10 µg/m3 (2 bytes)</th><th width="128">Checksum</th><th></th><th></th><th></th><th><select></select></th></tr></thead><tbody><tr><td>0x81</td><td>0x11</td><td>0x00</td><td>0x022B</td><td>0x06F4</td><td>0x06F4</td><td>0x0A82</td><td>0x1FC6</td><td>0x1FC6</td><td>0xF7</td><td></td><td></td><td></td><td></td></tr><tr><td>0x81</td><td>0x12</td><td>0x00</td><td>0x022B</td><td>0x06F4</td><td>0x06F4</td><td>0x0A82</td><td>0x1FC6</td><td>0x1FC6</td><td>0xF6</td><td></td><td></td><td></td><td></td></tr><tr><td>0x81</td><td>0x13</td><td>0x00</td><td>0x022B</td><td>0x06F4</td><td>0x06F4</td><td>0x0A82</td><td>0x1FC6</td><td>0x1FC6</td><td>0xF5</td><td></td><td></td><td></td><td></td></tr><tr><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr></tbody></table>

We advise the user to always read the sensor state, it highlights the functional state of the NextPM and allows to know the validity of the measured values.

Example : the frame response 0x81 0x12 0x00 0x00 0x0D 0x00 0x0E 0x00 0x0F 0x00 0x6A 0x00 0x72 0x00 0x85 0xE2 means that the results are averaged over 1 minute and no error has occurred during the measurement.

In this example, the measured concentrations are:

| Description | Data (2 bytes) | Factor | Results |
| ----------- | -------------- | ------ | ------- |
| PM1 Nb/mL   | 0x000D         | 1      | 13      |
| PM2.5 Nb/mL | 0x000E         | 1      | 14      |
| PM10 Nb/mL  | 0x000F         | 1      | 15      |
| PM1 µg/m3   | 0x006A         | 0.1    | 10.6    |
| PM2.5 µg/m3 | 0x0072         | 0.1    | 11.4    |
| PM10 µg/m3  | 0x0085         | 0.1    | 13.3    |



* 2.2.2.2           0x25, 0x26, 0x27 replies

<table><thead><tr><th width="99">Address</th><th width="97">Cmd code</th><th width="124">State (1 byte)</th><th>Channel 0.3 - 0.5 μm (in number of particles per Liter of air)</th><th>Channel 0.5 - 1 μm (in number of particles per Liter of air)</th><th>Channel 1 - 2.5 μm (in number of particles per Liter of air)</th><th>Channel 2.5 - 5 μm (in number of particles per Liter of air)</th><th>Channel 5 - 10 μm (in number of particles per Liter of air)</th><th>Checksum</th></tr></thead><tbody><tr><td>0x81</td><td>0x25</td><td>0x00</td><td>0x0004 0x9233</td><td>0x0000 0x1259</td><td>0x0000 0x0379</td><td>0x0000 0x0000</td><td>0x0000 0x0000</td><td>0xF7</td></tr><tr><td>0x81</td><td>0x26</td><td>0x00</td><td>0x0004 0xA424</td><td>0x0000 0x0F69</td><td>0x0000 0x03C3</td><td>0x0000 0x00B5</td><td>0x0000 0x0000</td><td>0x9A</td></tr><tr><td>0x81</td><td>0x27</td><td>0x00</td><td>0x0004 0x9117</td><td>0x0000 0x0FA3</td><td>0x0000 0x0439</td><td>0x0000 0x0078</td><td>0x0000 0x0013</td><td>0x32</td></tr></tbody></table>

We advise the user to always read the sensor state, it highlights the functional state of the NextPM and allows to know the validity of the measured values.\
\
Example : the frame response 0x81 0x27 0x00 0x00 0x04 0x91 0x17 0x00 0x00 0x0F 0xA3 0x00 0x00 0x04 0x39 0x00 0x00 0x00 0x78 0x00 0x00 0x00 0x13 0x32 means that the results are averaged over 15 minutes and no error has occurred during the measurement.

In this example, the measured concentrations are:

| Description                                                    | Data MSB (2 bytes) | Data LSB (2 bytes) | Results |
| -------------------------------------------------------------- | ------------------ | ------------------ | ------- |
| Channel 0.3 - 0.5 μm (in number of particles per Liter of air) | 0x0004             | 0x9117             | 299 287 |
| Channel 0.5 - 1 μm (in number of particles per Liter of air)   | 0x0000             | 0x0FA3             | 4003    |
| Channel 1 - 2.5 μm (in number of particles per Liter of air)   | 0x0000             | 0x0439             | 1081    |
| Channel 2.5 - 5 μm (in number of particles per Liter of air)   | 0x0000             | 0x0078             | 120     |
| Channel 5 - 10 μm (in number of particles per Liter of air)    | 0x0000             | 0x0013             | 19      |



* **2.2.2.3           0x14 command reply**

| Address | Command code | State | Temperature | Humidity | Checksum |
| ------- | ------------ | ----- | ----------- | -------- | -------- |
| 0x81    | 0x14         | 0x00  | 0x0B40      | 0x13E7   | 0x26     |

The raw temperature and relative humidity values are sent on 2 bytes and must be divided by 100 to get the physical values in °C and %RH.

Example : Here, the NextPM replies the temperature raw value of 0x0B40 0x13E7 which is 2880 in decimal. After dividing by 100, the physical value is 28.80 °C. Same calculation should be applied for calculating physical relative humidity.



* **2.2.2.4          0x16 command reply (state)**

| Address | Cmd code | State (1 byte) | Checksum |
| ------- | -------- | -------------- | -------- |
| 0x81    | 0x16     | 0x04           | 0x65     |

The 0x16 command frame requests the State code. This reply is also the default response to other requests when the NextPM has no data to send neither because the sensor has just been switched on nor because the sensor is in the Default State and/or Sleep State. See [Status code description](nextpm-user-guide.md#status-code-description) for State code decoding.



* **2.2.2.5          0x17 command reply**

| Address | Cmd code | State | Firmware version | Checksum |
| ------- | -------- | ----- | ---------------- | -------- |
| 0x81    | 0x17     | 0x00  | 0x0034           | 0x34     |

The NextPM replies to its firmware version. If the NextPM is in the SLEEP mode, it will only reply to its state code.



* **2.2.2.5**          0x01 Id reply

| Address | Cmd code | State | Id number          | Batch Number  | Checksum |
| ------- | -------- | ----- | ------------------ | ------------- | -------- |
| 0x81    | 0x01     | 0x00  | 0x0000&#xD; 0x0489 | 0x0000 0x8727 | 0x43     |

The NextPM replies the Id number + Batch number attributed. The pair of numbers represent a unique combination attributed to each NextPM sensor.\
Here the Id number is 0x0489 → 1161 (in decimal)\
Here the Batch number is 0x8727 → 34599 (in decimal)

#### 2.2.3 Writing command list <a href="#writing-command-list" id="writing-command-list"></a>

| Cmd code | Description                 | full command        |
| -------- | --------------------------- | ------------------- |
| 0x15     | Sleep mode entry / exit (1) | 0x81 0x15 0x6A      |
| 0x22     | Modbus address write (2)    | 0x81 0x22 0x03 0x5A |
| 0x41     | Heater OFF (0%)             | 0x81 0x41 0x3E      |
| 0x42     | Heater ON (100%)            | 0x81 0x42 0x3D      |
| 0x43     | Automatic heater regulation | 0x81 0x43 0x3C      |

(1) At each 0x15 command frame sent, the NextPM changes its functional state alternately. To know its state (see [Status code description](nextpm-user-guide.md#status-code-description)) before sending the command frame, the user can send a 0x16 command frame

(2) The 0x22 command is used to write the new modbus address. The third byte (here 0x03) is the address, which must be different from 0. See [0x22 command reply](nextpm-user-guide.md#writing-replies).



#### 2.2.4 Writing replies <a href="#writing-replies" id="writing-replies"></a>

* **2.2.4.1          0x15 command reply (sleep mode)**

| address | Command code | State (1 byte) | Checksum |
| ------- | ------------ | -------------- | -------- |
| 0x81    | 0x15         | 0x01           | 0x69     |

When the sensor receives a sleep mode command frame, it turns into sleep mode, stops its functionalities (fan, laser, heater and so, the measurements) and replies with a state code whose the sleep flag is set

During the sleep mode, if the sensor receives a new 0x15 command frame, the NextPM switches on and sends the first valid PM data after 15 seconds.

While the sensor is in sleep mode, it replies to all the other command frames by sending a 0x16 command code and the state code.

If the NextPM is in the Default State and so in sleep mode, the 0x15 command frame turns on the sensor: it allows the user to try a restart without errors.



* **2.2.4.2          0x22 command reply**

| address | Command code | State (1 byte) | Modbus address | Checksum |
| ------- | ------------ | -------------- | -------------- | -------- |
| 0x81    | 0x22         | 0x00           | 0x03           | 0x5A     |

The user can change the modbus device address using this command. The sensor replies to its new modbus address. The addresses start at 1 up to 15 (decimal values)



* **2.2.4.3          0x41, 0x42 and 0x43 command replies**

| address | Command code         | State (1 byte) | Checksum             |
| ------- | -------------------- | -------------- | -------------------- |
| 0x81    | 0x41 or 0x42 or 0x43 | 0x00           | 0x3E or 0x3D or 0x3C |

Those command codes allow the user to set a heater mode (stopped, full or auto-regulated).

Special care trying to modify the heater mode which is auto-regulated by default to improve accuracy in case of high relative humidity.



#### 2.2.5 Error codes <a href="#error-codes" id="error-codes"></a>

The NextPM sensor replies the state code (see [0x16 command reply (state)](nextpm-user-guide.md#reading-replies)) in these different cases:

* sleep mode
* default mode
* measurements not ready (in case of power on or sleep mode exit)

It won’t send back any response in case of:

* wrong checksum
* wrong command code
* wrong address
* command frame length too long (>5 bytes)





### 2.3 Modbus Protocol <a href="#modbus-protocol" id="modbus-protocol"></a>

The second way to get data from the NextPM sensor is using the modbus protocol, respecting the modbus standard (v1.1b modbus specification compliance).

#### 2.3.1 General information <a href="#general-information" id="general-information"></a>

* 2.3.1.1          Bytes order

All registers are coded on 2 bytes (word) but some data can be coded on 2 or more registers.

If a data is coded on 4 bytes (2 words, so 2 registers), such as PM measurements, the second word is the most significant one.

Example: 0x01 0x03 0x04 0x4E 0x10 0x00 0x03 0xAC 0xDF, is the reply of PM2.5 measurement value request. The PM2.5 raw value is: (0x4E10) + (0x0003)\*65536 = 216 592. Then, a factor must be applied to get the measurement in the correct unit (see [Readable registers list](nextpm-user-guide.md#readable-registers-list))

* 2.3.1.2          Modbus default address

Default modbus address is 0x01 but it can be changed by sending a simple protocol write command frame or modbus command write frame. The change is saved into its memory and is kept even after a restart.

* 2.3.1.3          Timings

Timings are the ones specified in [Serial communication configuration](nextpm-user-guide.md#serial-communication-configuration):

* The NextPM considers a timeout when there is 50ms of idle state between 2 bytes.
* The NextPM delay is 50 ms (beginning of the reply to a request).
* 2.3.1.4          Function codes

NextPM registers are accessible through 3 function codes:

* 0x03: read multiple holding registers
* 0x10: write multiple registers
* 0x17: Read/Write Multiple registers



#### 2.3.2 Readable registers list <a href="#readable-registers-list" id="readable-registers-list"></a>

_The following registers are only readable and accessible with a read or read/write multiple holding registers function command (0x03 or 0x17)_

| Register                         | Name                                           | Description                                                                                                                |
| -------------------------------- | ---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| 1 (0x01)                         | Firmware version                               | Embedded firmware version                                                                                                  |
| 19 (0x13)                        | Status                                         | NextPM status (error codes)                                                                                                |
| 109-110-111 (0x6D - 0x6E - 0x6F) | PM10 clogging                                  | Mass of PM10 cumulated by the sensor (divide by 10 to get the mass in milligrams, the maximum threshold is fixed to 12 mg) |
| 50-51 (0x32 - 0x33)              | PM1 10 sec average (Nb/L)                      | average over 10s of particles quantity per liter whose size is < 1μm (1)                                                   |
| 52-53 (0x34 - 0x35)              | PM2.5 10 sec average (Nb/L)                    | average over 10s of particles quantity per liter whose size is < 2.5μm (1)                                                 |
| 54-55 (0x36 - 0x37)              | PM10 10 sec average (Nb/L)                     | average over 10s of particles quantity per liter whose size is < 10μm (1)                                                  |
| 56-57 (0x38 - 0x39)              | PM1 10 sec average (µg/m3)                     | average over 10s of particles mass concentration (µg/m3) whose size is < 1μm (1) (2)                                       |
| 58-59 (0x3A - 0x3B)              | PM2.5 10 sec average (µg/m3)                   | average over 10s of particles mass concentration (µg/m3) whose size is < 2.5μm (1) (2)                                     |
| 60-61 (0x3C - 0x3D)              | PM10 10 sec average (µg/m3)                    | average over 10s of particles mass concentration (µg/m3) whose size is < 10μm (1) (2)                                      |
| 62-63 (0x3E - 0x3F)              | PM1 1 min average (Nb/L)                       | average over 1 min of particles quantity per liter whose size is < 1μm (1)                                                 |
| 64-65 (0x40 - 0x41)              | PM2.5 1 min average (Nb/L)                     | average over 1 min of particles quantity per liter whose size is < 2.5μm (1)                                               |
| 66-67 (0x42 - 0x43)              | PM10 1 min average (Nb/L)                      | average over 1 min of particles quantity per liter whose size is < 10μm (1)                                                |
| 68-69 (0x44 - 0x45)              | PM1 1 min average (µg/m3)                      | average over 1 min of particles mass concentration (µg/m3) whose size is < 1μm (1) (2)                                     |
| 70-71 (0x46 - 0x47)              | PM2.5 1 min average (µg/m3)                    | average over 1 min of particles mass concentration (µg/m3) whose size is < 2.5μm (1) (2)                                   |
| 72-73 (0x48 - 0x49)              | PM10 1 min average (µg/m3)                     | average over 1 min of particles mass concentration (µg/m3) whose size is < 10μm (1) (2)                                    |
| 74-75 (0x4A - 0x4B)              | PM1 15 min average (Nb/L)                      | average over 15 min of particles quantity per liter whose size is < 1μm (1)                                                |
| 76-77 (0x4C - 0x4D)              | PM2.5 15 min average (Nb/L)                    | average over 15 min of particles quantity per liter whose size is < 2.5μm (1)                                              |
| 78-79 (0x4E - 0x4F)              | PM10 15 min average (Nb/L)                     | average over 15 min of particles quantity per liter whose size is < 10μm (1)                                               |
| 80-81 (0x50 - 0x51)              | PM1 15 min average (µg/m3)                     | average over 15 min of particles mass concentration (µg/m3) whose size is < 1μm (1) (2)                                    |
| 82-83 (0x52 - 0x53)              | PM2.5 15 min average (µg/m3)                   | average over 15 min of particles mass concentration (µg/m3) whose size is < 2.5μm (1) (2)                                  |
| 84-85 (0x54 - 0x55)              | PM10 15 min average (µg/m3)                    | average over 15 min of particles mass concentration (µg/m3) whose size is < 10μm (1) (2)                                   |
| 100 (0x64)                       | Fan command ratio (%)                          | Fan command ratio (%) (to be divided by 100)                                                                               |
| 101 (0x65)                       | Heater command ratio (%)                       | Heater command ratio (%) (to be divided by 100)                                                                            |
| 102 (0x66)                       | Fan speed                                      | Fan rotation speed (Hz)                                                                                                    |
| 103 (0x67)                       | Laser status                                   | 1 for laser on 0 for laser off                                                                                             |
| 106 (0x6A)                       | Relative humidity (internal)                   | Relative humidity in % (to be multiplied by 100)                                                                           |
| 107 (0x6B)                       | Temperature (internal)                         | Temperature in °C (to be multiplied by 100)                                                                                |
| 128-129 (0x80 - 0x81)            | 0.2µm to 0.5µm quantity (Nb/L) 10 sec average  | average over 10s of particles quantity per liter whose size is between 0.2µm and 0.5µm (1)                                 |
| 130-131 (0x82 - 0x83)            | 0.5µm to 1.0µm quantity (Nb/L) 10 sec average  | average over 10s of particles quantity per liter whose size is between 0.5µm to 1.0µm (1)                                  |
| 132-133 (0x84 - 0x85)            | 1.0µm to 2.5µm quantity (Nb/L) 10 sec average  | average over 10s of particles quantity per liter whose size is between 1.0µm to 2.5µm (1)                                  |
| 134-135 (0x86 - 0x87)            | 2.5µm to 5.0µm quantity (Nb/L) 10 sec average  | average over 10s of particles quantity per liter whose size is between 2.5µm to 5.0µm (1)                                  |
| 136-137 (0x88 - 0x89)            | 5.0µm to 10.0µm quantity (Nb/L) 10 sec average | average over 10s of particles quantity per liter whose size is between 5.0µm to 10.0µm (1)                                 |
| 145 (0x91)                       | External calculated Temperature (°C)           | Temperature in °C (to be multiplied by 100)                                                                                |
| 146 (0x92)                       | External calculated Relative humidity (%)      | Relative humidity in % (to be multiplied by 100)                                                                           |

1. _See example below explaining how to decode mass and quantity concentration._
2. _Mass concentration should be divided by 1000 to get the value in µg/m3_



Measurement decoding example :

* PM1, PM2.5 and PM10 concentrations request command: 01 03 00 32 00 24 E4 1E
*   Sensor’s reply :

    01 03 48 62 4F 00 25 62 4F 00 25 62 4F 00 25 00 EC 00 00 00 EC 00 00 00 EC 00 00 6A 5D 00 13 99 6F 00 14 57 22 00 15 00 5E 00 00 01 82 00 00 03 A8 00 00 00 ED 00 17 CA FA 00 17 FE 29 00 17 00 A7 00 00 01 C8 00 00 02 69 00 00 77 09

    * Calculation of the 10s average of the particles number concentration / liter whose size is < 1μm :
      * 0x624F is the less significant word and 0x0025 is the most significant word
      * the concatenate value (raw value) is 0x0025624F which is 2449999 in decimal. This is the measured physical value in Nb/L, averaged over 10 seconds.
    * Calculation of the 10s average of the mass concentration of particles whose size is < 1μm :
      * 0x00EC is the less significant word and 0x0000 is the most significant word
      * the concatenate value (raw value) is 0x000000EC which is 236 in decimal
      * Measured physical value of PM1 is 0.236 µg/m3 , averaged over 10 seconds.



2.3.3 Writable registers list

_The following registers are writable and accessible with a write or read/write multiple holding registers function command (0x10 or 0x17). These are advanced registers that need to be considered carefully before changed, as they may modify the sensor behaviour. We recommend contacting Tera Sensor before modifying them._

| Register   | Name                                               | Description                                                                                                                                                                       |
| ---------- | -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 88 (0x58)  | Modbus address                                     | NextPM with Modbus protocol ( range is 1 to 15)                                                                                                                                   |
| 100 (0x64) | Fan target ratio (%)                               | Fan rotation set point (%) (to be divided by 100) (1)                                                                                                                             |
| 101 (0x65) | Heater ratio (%)                                   | Cyclic ratio for heating (expressed in %) (to be divided by 100) (1) 0000: turn off the heater (manually) 2710: turn on the heater (manually) FFFF: auto regulation of the heater |
| 103 (0x67) | Laser function                                     | 0: turn off the laser 1: turn on the laser other values (auto regulation of the laser)                                                                                            |
| 112 (0x70) | Baud rate                                          | Identified code: range is 1 to 8 (2)                                                                                                                                              |
| 104 (0x68) | Validation Key for writing in registers 105 to 110 | The key value is 0x33 0xAA and cannot be changed                                                                                                                                  |
| 105 (0x69) | Coefficient for PM1                                | Slope coefficient initially set to 100 (0x 00 0x64) because factor 0.01 is applied to get 2 decimal values                                                                        |
| 106 (0x6A) | Off set for PM1                                    | Off set coefficient initially set to 10 000 (0x27 0x10) because factor 0.01 is applied first, and -100 is applied secondly                                                        |
| 107 (0x6B) | Coefficient for PM2.5                              | Slope coefficient initially set to 100 (0x64) because factor 0.01 is applied to get 2 decimal values                                                                              |
| 108 (0x6C) | Off set for PM1                                    | Off set coefficient initially set to 10 000 (0x27 0x10) because factor 0.01 is applied first, and -100 is applied secondly                                                        |
| 109 (0x6D) | Coeffi cient for PM10                              | Slope coefficient initially set to 100 (0x64) because factor 0.01 is applied to get 2 decimal values                                                                              |
| 110 (0x6E) | Off set for PM1                                    | Off set coefficient initially set to 10 000 (0x27 0x10) because factor 0.01 is applied first, and -100 is applied secondly                                                        |

1. As it will change the behavior of the sensor and the reliability of the measurements, Tera Sensor does not recommend any change .
2. The baud rate of the simple and modbus communication can be changed by writing the identifier code corresponding to the desired baud rate value (see table below).

| Identifier code | Baud rate |
| :-------------: | :-------: |
|        1        |   115200  |
|        2        |   57600   |
|        3        |   38400   |
|        4        |   19200   |
|        5        |    9600   |
|        6        |    4800   |
|        7        |    2400   |
|        8        |    1200   |

Special care should be taken when Baud rate is changed, the user should mark or remember the new baud rate associated with the Next-PM because if it is forgotten by the user, there is no way to find it except by testing all the baud rate values up to match the baud rate previously programmed.

About coefficient/off set correction for PM mentioned above at registers 105 to 110. An example of modification is given below, with the explanation of the final coefficient and off set obtained.

Frame example to change coefficient a (at register 105) for PM1 at 1.16 and the off set b (at register 106 at 0.62, but to maintain coefficients and off sets as initial values for PM2.5 and PM10 :&#x20;

0x01 0x10 0x00 0x68 0x00 0x07 0x0E **0x33 0xAA** 0x00 0x74 **0x27 0x4E** 0x00 0x64 0x27 0x10 **0x00 0x64 0x27 0x10** 0xF9 0x58&#x20;

Frame details and calculation for Coefficients and Off sets&#x20;

0x01 0x10 0x00 0x68 0x00 0x07 0x0E : Modbus starting answer&#x20;

**0x33 0xAA** : Key to modify the coefficients and off sets values&#x20;

0x00 0x74 : Coefficient for PM1 at 116 (final value 116/100 = 1.16)&#x20;

**0x27 0x4E** : Off set for PM1 at 10 062 (final value \[10 062/100] -100 = 0.62)&#x20;

0x00 0x64 0x27 0x10 : Initial Coefficient and Off set for PM2.5 set as 1 and 0 respectively&#x20;

**0x00 0x64 0x27 0x10** : Initial Coefficient and Off set for PM10 set as 1 and 0 respectively&#x20;

0xF9 0x58 : CRC&#x20;

This way of calculating the Coefficient and Off set allows values:&#x20;

\- For coefficient to vary from 0 to 100 (as final value already divided by 100)&#x20;

\- For off set to vary from -100 to +100 (as final value already divided by 100 and subtract of 100)&#x20;

If you may have any doubts calculating the coefficients and off set for your calibration, please contact support@groupe-tera.com.



#### 2.3.2 Error codes <a href="#readable-registers-list" id="readable-registers-list"></a>

The NextPM sensor is able to make specifi c reply depending on the Modbus frame received:

\
■ Error code, with specifi c error code depending on the Modbus frame received:\
&#x20;         ○ Inexistant code function (CF)\
&#x20;                   \-  reply: ModbusAdress 0x80|CF 0x01 CRC1 CRC2\
&#x20;         ○ Inexistant registers (write)\
&#x20;                   \- reply: ModbusAdress 0x80|CF 0x02 CRC1 CRC2 note: only true for 0x10 code function. For the 0x17 code function, the NextPM sensor sends back correctly the data read, without any error code.

\
■ No reply, in case of:\
&#x20;         ○ wrong checksum\
&#x20;         ○ wrong Modbus address

\
■ Standard reply, even if the frame received is partially correct:\
&#x20;         ○ Registers quantity exceeding the maximum allowed:\
&#x20;                   \- 125 registers to be read in a single time\
&#x20;                   \- 18 registers to be written in a single time\
In those cases, the NextPM sensor will set the quantity requested equal to the maximum. 19\
&#x20;         ○  nonexistent registers (to be read)\
&#x20;                   -data corresponding to the nonexistent register will be 0x00 0x00\
&#x20;         ○  Frame content inconsistency (number of bytes not as expected depending on the code function):\
&#x20;                 -If there are more bytes than expected, the NextPM sensor truncates the frame to match the size with the one expected according to the code function. It will then interpret the truncated frames and the reply could not be consistent with the user request.\
&#x20;                 -If there are less bytes than expected, the NextPM sensor interprets the frames and the reply could not be consistent with the user request.

***



## 3. RS 485

If the user wants to integrate the NextPM on an RS485 half duplex bus, despite the NextPM does not have an embedded RS485 transceiver, it has the ability to drive an external one thanks to its DE/RE output signal which indicates to the RS485 transceiver if it should be in receive or in transmission mode.

Even if any module of that kind could work also well, Tera Sensor is used to work with the PModRS485 (from digilent supplier) or the DSD TECH SH-U12. Both converters has been tested and work properly for that purpose.



***



## 4. Test software <a href="#test-software" id="test-software"></a>

Tera Sensor provides software that can help the user to communicate with the sensor and test it before working on the integration.

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 110356.png" alt=""><figcaption></figcaption></figure>

Note: The computer running the software should have Windows 10 or newer version.



***



## 5. NextPM Mechanical tips <a href="#nextpm-mechanical-tips" id="nextpm-mechanical-tips"></a>



### 5.1 Introduction <a href="#id-5.1-introduction" id="id-5.1-introduction"></a>

This document has been written in order to support our customer to optimize the integration of the NextPM Sensor in their device and obtain the best and most accurate results for PM monitoring. The mechanical settings and the guidelines for integration of the NextPM will be approached in the following part of the document to guarantee the best performance for the sensor. This guideline is a tips summary of more than 20 years experience in the air quality monitoring domain.



### 5.2 Mechanical settings <a href="#id-5.2-mechanical-settings" id="id-5.2-mechanical-settings"></a>

#### 5.2.1 Air path in the sensor <a href="#id-5.2.1-air-path-in-the-sensor" id="id-5.2.1-air-path-in-the-sensor"></a>

The NextPM Sensor presents a 6 pins Molex connector to ensure power supply and to communicate with the sensor. Once the power supply is provided to the sensor, the sensor turns on automatically, the fan starts and the airflow path operates. The air inlet is facing the front of the sensor, while the outlet is rejecting the air by the side of the sensor to limit PM recycling effects.

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 110440.png" alt=""><figcaption></figcaption></figure>

#### 5.2.2 Fixation <a href="#id-5.2.2-fixation" id="id-5.2.2-fixation"></a>

You can fix the NextPM sensor thanks to the two holes in the back of the sensor. The holes are made to receive a tap screws M2.5 and are 5 mm deep.

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 110514.png" alt=""><figcaption></figcaption></figure>



***



## 6. General NextPM integration <a href="#id-6.-general-nextpm-integration" id="id-6.-general-nextpm-integration"></a>

### 6.1 Positions for storage <a href="#id-6.1-positions-for-storage" id="id-6.1-positions-for-storage"></a>

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 110546.png" alt=""><figcaption></figcaption></figure>

### 6.2 Positions to operate <a href="#id-6.2-positions-to-operate" id="id-6.2-positions-to-operate"></a>

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 110612.png" alt=""><figcaption></figcaption></figure>

Why this recommendations for operating conditions:

* The NextPM inlet is an aeraulic filter that stops particles larger than 10µm from entering the sensor. If the sensor is upside down, then the aeraulic filter will not be able to block those big particles and the lifetime of the sensor will be affected due to clogging.
* The fan is a critical part if not correctly set in the sensor. Inside the fan, a small amount of oil allows the rotation without friction. If the NextPM (and consequently the fan) is not correctly placed, the oil can leak and it results with stretching in the fan, an increased power consumption to compensate for the frictions, until the failure of the fan. That will strongly affect the lifetime of the sensor.



### 6.3 Pressure lost and Deported Inlet <a href="#id-6.3-pressure-lost-and-deported-inlet" id="id-6.3-pressure-lost-and-deported-inlet"></a>

#### 6.3.1 Pressure Loss

The sensor uses an active airflow to sample the particles. In order to keep his accuracy, we advise you to put the inlet directly to the air you want to monitor. Be aware that the fan allowing the flow inside the sensor is sensitive to pressure loss, it does not allow a differential pressure between the inlet and outlet more than 0.3 mbar. As the fluidic path already generates a 0.15 mbar pressure loss, the remaining possible pressure loss is 0.15 mbar due to additional features, for instance a deported inlet of the NextPM.

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 110641.png" alt=""><figcaption></figcaption></figure>

#### 6.3.2 Deported Inlet

If the integration is difficult, you can add a duct made with antistatic materials and with an internal diameter of 8 to 10 mm (depending on the rigidity of the material). The maximum length of the duct must not exceed 100 cm to avoid particle losses in the tube, and approximative measurements. Avoid sharp angles in the tubing as much as possible to prevent particle losses and approximation in measurements.

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 110711.png" alt=""><figcaption></figcaption></figure>

### 6.4 Inlet protection <a href="#id-6.4-inlet-protection" id="id-6.4-inlet-protection"></a>

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 110737.png" alt=""><figcaption></figcaption></figure>

The NextPM sensor is an IP20 device. The sensor must be protected from direct water droplets such as rain or cleaning tools.

To extend the lifetime of the device, we recommend placing a metallic grid at the inlet of the NextPM to avoid insects or dust getting in the sensor. A cap including the grid can also be provided by Tera Sensor to correctly set the grid at the NextPM inlet (please contact [support@groupe-tera.com](mailto:support@groupe-tera.com) for further details).

### 6.5 Sensor integration <a href="#id-6.5-sensor-integration" id="id-6.5-sensor-integration"></a>

The NextPM main purpose is to monitor the particulate matter. Different types of environments or applications are possible for the sensor. For example,

The NextPM is designed for outdoor PM monitoring, but can also be used for indoor uses. If you plan to use the sensor outdoors, an integration to a casing is needed to maintain the performance of the NextPM sensor (not waterproof). There are principal cases:

* Open casings
* HVAC Sealed casing

#### 6.5.1 Open casings <a href="#id-6.5.1-open-casings" id="id-6.5.1-open-casings"></a>

* The louver system

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 110829.png" alt=""><figcaption></figcaption></figure>

The casing with louver is a good option for outdoor air PM monitoring. It allows a good ventilation of the air inside the casing to be sure that the pollution is the same inside the casing and outside. It prevents the NextPM sensor from rain, and does not expose the sensor to direct sunlight. Moreover, the inlet and outlet are at the same pressure.\
A cap including the grid can also be provided to set correctly the grid at the NextPM inlet and avoid large dust particles or small insects to get in the sensor.\
We recommend keeping at least 4 centimeters distance between the NextPM inlet and outlet from the louver walls.

* The bottom open systems

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 110858.png" alt=""><figcaption></figcaption></figure>

A bottom open system is also possible for the NextPM integration. The larger the opening, the better the sensor will operate.

A gap between the sensor and the wall of at least 2 cm is recommended to avoid obstruction of the inlet and outlet of the sensor.

Those two cases presented above are not the only ones possible and will depend on every integrator regarding the new or existing product to integrate the NextPM. If doubts are present about the integration of the sensor, please feel free to contact us at [support@groupe-tera.com](mailto:support@groupe-tera.com) in order to discuss and provide the best solutions for a good integration.

#### 6.5.2 Sealed casing <a href="#id-6.5.2-sealed-casing" id="id-6.5.2-sealed-casing"></a>

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 110925.png" alt=""><figcaption></figcaption></figure>

This casing is more dedicated to sampling particles for dedicated environments, such as HVAC systems directly in ventilation tubings. The Inlet tubing must be antistatic to avoid particle losses.

The outlet connection must be connected to the same location of the inlet to avoid pressure losses that could not be tolerated by the NextPM.

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 111517.png" alt=""><figcaption></figcaption></figure>

#### 6.5.3 HVAC <a href="#id-6.5.3-hvac" id="id-6.5.3-hvac"></a>

The NextPM has been developed first to be integrated in cars where it could be exposed to different air velocities. Therefore, we studied what will be the behavior of the sensor if we change the direction and the speed of air. The results could be used in every use case where the air speed is not still.

The first conclusion of the study is that the sensor has better performances if it is not integrated in an additional standard casing. The second one is that the air direction has more impact than the air speed itself.

Therefore, when you integrate the NextPM sensor to monitor the air quality into a HVAC system, please respect the following placement of NextPM sensor into the airflow:

Be aware that it is not recommended to place the inlet of the NextPM in an opposite direction with the airflow of the HVAC system.

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 111614.png" alt=""><figcaption></figcaption></figure>

If you can’t place the sensor into the airflow directly, the main concern you have to mind is that the inlet and the outlet of the NextPM must be set at the same pressure. The sealed casing presented in the general integration tips can be used.

### 6.6 Light exposure <a href="#id-6.6-light-exposure" id="id-6.6-light-exposure"></a>

Direct light exposure can modify the sensor performance. For example the temperature measurements can be affected by the heating process due to direct sunlight exposure. Moreover, if the sensor is directly exposed to light the PM measurements can be affected due to light entering the sensor by the outlet. This will generate measurement artifacts due to the light beam arriving at the photodetectors. This is why it is always recommended to turn the sensor to the North direction in the case of outdoor uses.

The following picture presents how to integrate the NextPM sensor into a casing dedicated to outdoor measurements, preventing the sensor from light exposure, and also environmental parameters.

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 111639.png" alt=""><figcaption></figcaption></figure>





***



## 7. Sequential measurement for Autonomous Device <a href="#id-7.-sequential-measurement-for-autonomous-device" id="id-7.-sequential-measurement-for-autonomous-device"></a>

The NextPM sensor is a good option for autonomous products, but there are conditions to respect in order to obtain the best results possible.

The NextPM sensor is equipped with a sleep mode to reduce power consumption, or it is also possible to add a power switch to turn the sensor completely off (depending on the maximum power consumption possible).

After an off/sleep period, the sensor is turned back on, it is needed to wait 15 seconds before the sensor is fully operational (mainly for the fluidic path to become stable after the fan restart) . It is also recommended to wait an additional 20 seconds minimum (1 minute is better) before collecting the data from the 1 minute average command for PM1, PM2.5 and PM10.

In this case the results obtained will be reliable and can be compared with other instruments if needed.

Table resuming the example cited above:

|             |    Cycle description    |                                                  |                                            |
| :---------: | :---------------------: | :----------------------------------------------: | :----------------------------------------: |
| Cycle Steps | NextPM off / sleep mode |             NextPM on (measuring PM)             |                                            |
| Description |   Sensor not measuring  | Warm up period (NextPM on but not measuring yet) | Data collection (1 minute average channel) |
|   Timeline  |    As long as needed    |                      15 sec.                     |              20-60 sec. later              |

**Special cases:**

In specific environmental conditions, the sensor recovering from the sleep mode will take more time, for instance for very low temperatures below 0°C.

Another classical scenario is if relative humidity is elevated (above 70%), then the heater function will need more than 1 minute to be fully operational.

In those particular cases, it is recommended to directly reach the Tera Sensor Support team at [support@groupe-tera.com](mailto:support@groupe-tera.com) to give a complete description of the configuration in order to have the optimized cycles for the NextPM sensor.



***



## 8. Electrical integrity <a href="#id-8.-electrical-integrity" id="id-8.-electrical-integrity"></a>

As the EMC shield of the sensor is connected to its ground, the integrator should avoid a mechanical contact to the system earth / ground, by using a non conductive material or an electrical insulation.

This will avoid any unwanted current path flow in case of power supply isolation fault, which would damage the sensor.





***



## 9. Accessories <a href="#id-9.-accessories" id="id-9.-accessories"></a>

Tera Sensor sells additional components to help the user:

* grid to avoid insect penetration (sensor protection)
* grid plugging system (easier integration)
* anti-static tube for remote short distance measuring (easier integration)
* plug and play: USB (FTDI) communication cable (plug and play for a quick use)\
  Note: it also power supplies the NextPM sensor through the USB power lines but special care must be taken regarding the current drawn from the USB port.
* Demo software to get data without getting into the code writing process (quick use)

Please visit [https://tera-sensor.com/fr/](https://tera-sensor.com/fr/) to find more information and to contact the sales service.





***



## 10. Revision history <a href="#id-10.-revision-history" id="id-10.-revision-history"></a>

| Date       | Revision | Changes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ---------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2018/05/12 | 2.2      | - Add warnings at serial port communication chapter - Add modbus protocol                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| 2019/11/04 | 3.0      | - Add summary - Remove GetTimeOn modbus reading command                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 2019/11/22 | 3.1      | - Remove fan speed write simple protocol                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| 2020/04/27 | 3.2      | -user guide format change -Remove STOP mode description as non existing anymore                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 2020/06/04 | 3.3      | -Change PM units for quantity per volume for simple protocol reading (pcs/mL to pcs/L) -Remove links of website and software download                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| 2020/10/21 | 3.4      | - Add additional information about NextPM integration advices (orientation)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| 2021/02/10 | 3.5      | - Add additional information about modbus NextPM available function codes - Put fan speed read command into correct corresponding chapter                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| 2021/04/02 | 3.6      | - Add baud rate modbus write command description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| 2024/08/30 | 4.0      | - user guide redesign and restructuring - Add [Accessories](broken-reference) chapter, add 5 channels quantity modbus readable registers. - Add Mechanical tips chapter - Add Sequential measurements chapter - Add Electronic integrity chapter - Correct delay value, modbus address write simple protocol frame example content, remove simple protocole FAN speed write command, correct modbus PM reading reply decoding example, correct PM units for quantity per volume for simple protocol reading (pcs/L to pcs/mL), correct PM quantity decoding example for modbus protocol |
| 2024/11/06 | 4.2      | <p>New sections add for the new fi rmware 1047:<br>-In Simple protocol communication, new commands for serial number, 5 channels particles number requests (on every time average 10s, 1min and 15min)<br>-In Mobus protocol communication, new commands for serial number, 5 channels particles number requests (on every time average 10s, 1min and 15min) and the possibility to modify coeffi cients and off set for PM measurements correction by the end user.</p>                                                                                                              |
