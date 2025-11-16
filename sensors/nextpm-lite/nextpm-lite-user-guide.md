---
icon: lightbulb-exclamation-on
---

# NextPM Lite User Guide

## 1. About NextPM Lite <a href="#id-3xi6axjdh0m0" id="id-3xi6axjdh0m0"></a>

### 1.1 Generals <a href="#id-7cy7l5u4hwdz" id="id-7cy7l5u4hwdz"></a>

The NextPM Lite sensor is a particulate matter measurement device which is able to detect PM1, PM2.5 and PM10 in mass concentration (µg/m3) and quantity (Nb/mL). All those data are available within 3 different average periods (10sec, 1 min and 10 min) to best suit the dynamic of the external environment.

It is also possible to measure particles classified in 5 different channels(1):

* 0.3µm to 0.5µm
* 0.5µm to 1µm
* 1µm to 2.5µm
* 2.5µm to 5µm
* 5µm to 10µm



Sensor specifications, such as range, precision, conditions of use, etc.., are defined into the datasheet of the NextPM Lite sensor which can be found at [https://tera-sensor.com](https://tera-sensor.com/)

The user can find explanations about fluidic and mechanics to be helped for its NextPM Lite use/integration by referring to the NextPM Lite integration advice chapter.

The Serial Communication available on the NextPM Lite are:

* simple
* modbus
* I2C



1. Only available since 0x0853 firmware version



### 1.2 Pinout and electrical considerations <a href="#bzyxeel0jse" id="bzyxeel0jse"></a>

<figure><img src="../../.gitbook/assets/3 (1).png" alt=""><figcaption></figcaption></figure>

#### 1.2.1 UART Pinout <a href="#id-5l2og5jf4ozl" id="id-5l2og5jf4ozl"></a>

| Pin # | Signal name                                |
| ----- | ------------------------------------------ |
| 1     | +5V                                        |
| 2     | CS (Communication Selector: not connected) |
| 3     | Tx (sensor output)                         |
| 4     | Rx (sensor input)                          |
| 5     | GND                                        |

#### 1.2.2 I2C Pinout <a href="#qnb7bm1u442d" id="qnb7bm1u442d"></a>

| Pin # | Signal name                                       |
| ----- | ------------------------------------------------- |
| 1     | +5V                                               |
| 2     | CS (Communication Selector: I2C): connected to 5V |
| 3     | SCL (Serial Clock Line)                           |
| 4     | SDA (Serial Data Line)                            |
| 5     | GND                                               |

* The NextPM Lite must be power supplied with + 5 VDC / 0.2V.
* The average current consumption is lower than 80mA . The average current consumption can be reduced to less than 5 mA by turning the sensor into sleep mode.
* The Tx and Rx signal logical level is TTL (+3.3V) and the Rx signal is +5V tolerant.
* The signal CS is useful only in case of I2C mode use, to drive the external RS485 transceiver, otherwise, it could be left unconnected (high impedance).
* The ADAM TECH _125CH-B-05_ or MOLEX _51021-0500_ connector can be purchased to connect the NextPM Lite Sensor..\
  To connect the NextPM Lite to a PC, a FTDI cable such as _TTL-232R-3V3 (supplier: FTDI chip)_ can be used.
* The serial communication is available thanks to PIN 3 and 4 and the NextPM Lite behaves as the slave during the communication, it will reply to command orders from the master.
* Note: 2 pull-up resistors should be mounted on each line (SDA and SCL). Their value should be chosen by the customer depending on the communication bus characteristics (speed rate, bus capacity, …). Typically, the standard value is 4.7kOhm.\


<mark style="color:red;">Warning : do not connect NextPM Lite sensor directly on the RS232 port of a PC or to a RS485 network.</mark>

### 1.3 Power saving <a href="#m3k7sb5ydm2z" id="m3k7sb5ydm2z"></a>

To save the system energy by reducing the average NextPM Lite current consumption, the user can put the NextPM Lite sensor into sleep mode by sending the corresponding command (see [Writing commands list](https://docs.google.com/document/d/1mDMpD21G-W-E4irnLSpVwDwx0-xLc66_a1_b9S5H0ZM/edit#heading=h.c4r5wogk8vuz) and Writable registers list). This reduces the current consumption below 5 mA.

In this mode, the NextPM Lite cannot provide any measurement as it has stopped the majority of its functionalities (fan and laser) but its communication remains functional.

To save more energy, if the user has the capability to, it is better to produce ON/OFF cycles by managing the sensor power supply . This way, the consumption will be 0mA during the NextPM Lite off phase and the average consumption will be considerably reduced.

### 1.4 Data availability <a href="#fhzv21o4wwv" id="fhzv21o4wwv"></a>

At the power on of the sensor or at the sleep mode exit, the measurements are reliable after 15 seconds.

### 1.5 Status code description <a href="#id-1oqkk0z3d70x" id="id-1oqkk0z3d70x"></a>

The NextPM Lite makes continuous internal diagnosis in order to let the user know the state of the sensor and so, the validity of the measurements.

The state is coded on 2 bytes (1) and it contents flags corresponding to errors which can occur:\


| Bit 15 | Bit 14 | Bit 13 | Bit 12 | Bit 11 | Bit 10 | Bit 9 | Bit 8                      |
| ------ | ------ | ------ | ------ | ------ | ------ | ----- | -------------------------- |
| -      | -      | -      | -      | -      | -      | -     | <p>Default</p><p>State</p> |

| Bit 7       | Bit 6  | Bit 5     | Bit 4  | Bit 3  | Bit 2     | Bit 1          | Bit 0       |
| ----------- | ------ | --------- | ------ | ------ | --------- | -------------- | ----------- |
| Laser Error | Unused | Fan Error | Unused | Unused | Not Ready | Degraded State | Sleep State |

* bit 0: set if the user has sent a sleep mode command or if the sensor is in default state (bit 8).\
  In simple protocol, the only authorized command still possible is a NextPM Lite state read.\

* bit 1: set each time a minor error is detected and confirmed. The corresponding minor error flag is set. In this state, the NextPM Lite can still measure data but with less accuracy.\
  The minor errors are the following:
  * bit 5: Fan Error: the fan speed is out of range but the fan is still working
  * bit 7: Laser Error: the photodetector is not detecting any particle for at least 240 secs resulting from a possible laser error.\

* bit 2: set at power on, meaning that the sensor is starting and that measurements are not yet reliable until the bit is clear. It lasts 15 seconds.\
  This bit is also set after a wake-up of the sensor (sleep mode exit)\

*   bit 8: set if the fan has stopped working, after 3 restart attempts, the sensor will switch to the default mode, set the default state flag meanwhile the degraded state flag is clear and the sensor goes into sleep mode. In the same way, if no particles are detected for a period exceeding 12 minutes, then the sensor goes into default state and into sleep mode.

    (1) The 8 most significant bits are only available using modbus protocol

## 2. NextPM Lite communication <a href="#id-17dp8vu" id="id-17dp8vu"></a>

The NextPM Lite sensor is able to either communicate using a [simple protocol](nextpm-lite-user-guide.md#lzhpfme2ur3) based only on proprietary UART frames or using [Modbus standard ](nextpm-lite-user-guide.md#id-2bn6wsx)protocol. It is also possible to use[ I2C protocol](nextpm-lite-user-guide.md#wq4uaqsn4ee) to communicate with the NextPM Lite.

For every communication protocol, the physical layer is TTL 3.3V logical levels.

### 2.1 Serial communication configuration (simple and modbus) <a href="#id-6bmxpzzcqzjx" id="id-6bmxpzzcqzjx"></a>

The serial communication must be configured as followed :

* Default baud rate : 115 200
* Number of data bits: 8
* Parity : even
* Number of stop bits: 1

Notes:

* The NextPM Lite considers a timeout when there is 50ms of idle state between 2 bytes.
* The NextPM Lite delay is 50 ms (beginning of the reply to a request).

### 2.2 Simple protocol <a href="#lzhpfme2ur3" id="lzhpfme2ur3"></a>

The simple protocol uses the following frame format:

* Reading:
  * Request: address (0x81) + command function code + checksum
  * Response: address (0x81) + command function code + data + checksum\

* Writing:
  * Request: address (0x81) + command function code + data + checksum
  * Response: address (0x81) + command function code + checksum

\
The checksum is calculated in such a way that the sum of all the frame bytes is equal to a multiple of 256 (0x100): checksum = 0x100 – MOD ((sum of the other bytes), 256).

Examples :

* 0x81 + 0x16 + 0x69 = 0x100
* 0x81 + 0x21 + 0x55 + 0x09 = 0x100
* 0x81 + 0x12 + 0x00 + 0x00 + 0x0D + 0x00 + 0x0E + 0x00 + 0x0F+ 0x00 + 0x6A + 0x00 + 0x72 + 0x00 + 0x85 + 0xE2 = 0x300

#### 2.2.1 Reading commands list <a href="#id-44sinio" id="id-44sinio"></a>

The NextPM Lite information’s reading can be realized through the following command frames:

| Cmd code | Description                                                                          | Command                 |
| -------- | ------------------------------------------------------------------------------------ | ----------------------- |
| 0x11     | Concentrations (µg/m3 and Nb/mL) averaged over 10 seconds and updated every 1 second | 0x81 0x11 0x6E          |
| 0x12     | Concentrations (µg/m3 and Nb/mL) averaged over 1 min and updated every 10 seconds    | 0x81 0x12 0x6D          |
| 0x13     | Concentrations (µg/m3 and Nb/mL) averaged over 10 min and updated every 1 minute     | 0x81 0x13 0x6C          |
| 0x15     | Power on or sleep mode                                                               | 0x81 0x15 0x6A (note 1) |
| 0x16     | Sensor state                                                                         | 0x81 0x16 0x69          |
| 0x17     | Firmware version                                                                     | 0x81 0x17 0x68          |
| 0x22     | Modbus address                                                                       | 0x81 0x22 0x5D          |

#### 2.2.2 Reading replies <a href="#id-2jxsxqh" id="id-2jxsxqh"></a>

**2.2.2.1           0x11, 0x12, 0x13 replies**

| Address | Cmd code | <p>State</p><p>(1 byte)</p> | <p>PM1</p><p>Nb/mL</p><p>(4 bytes)</p> | <p>PM2.5</p><p>Nb/mL</p><p>(4 bytes)</p> | <p>PM10 Nb/mL</p><p>(4 bytes)</p> | <p>PM1 µg/m3</p><p>(2 bytes)</p> | <p>PM2.5 µg/m3</p><p>(2 bytes)</p> | <p>PM10 µg/m3</p><p>(2 bytes)</p> | Checksum |
| ------- | -------- | --------------------------- | -------------------------------------- | ---------------------------------------- | --------------------------------- | -------------------------------- | ---------------------------------- | --------------------------------- | -------- |
| 0x81    | 0x11     | 0x00                        | 0x022B                                 | 0x06F4                                   | 0x06F4                            | 0x0A82                           | 0x1FC6                             | 0x1FC6                            | 0xF7     |
| 0x81    | 0x12     | 0x00                        | 0x022B                                 | 0x06F4                                   | 0x06F4                            | 0x0A82                           | 0x1FC6                             | 0x1FC6                            | 0xF6     |
| 0x81    | 0x13     | 0x00                        | 0x022B                                 | 0x06F4                                   | 0x06F4                            | 0x0A82                           | 0x1FC6                             | 0x1FC6                            | 0xF5     |

\
We advise the user to always read the sensor state, it highlights the functional state of the NextPM Lite and allows to know the validity of the measured values.

**NB: The actual version codes the PM1, PM2.5 and PM10 on 4 bytes, but it will change in the final version of the firmware (and accordingly in the next userguide version).**

Example : the frame response 0x81 0x12 0x00 0x00 0x00 0x24 0x18 0x00 0x00 0x24 0xA6 0x00 0x00 0x00 0x8E 0x00 0x07 0x00 0x0F 0x00 0x12 0xB1 means that the results are averaged over 1 minute and no error has occurred during the measurement.

In this example, the measured concentrations are:

| Description | Data (4 or 2 bytes) | Factor | Results |
| ----------- | ------------------- | ------ | ------- |
| PM1 Nb/L    | 0x0000 0x2418       | 1      | 9240    |
| PM2.5 Nb/L  | 0x0000 0x24A6       | 1      | 9382    |
| PM10 Nb/L\* | 0x0000 0x008E       | 1      | 15      |
| PM1 µg/m3   | 0x0007              | 0.1    | 0.7     |
| PM2.5 µg/m3 | 0x000F              | 0.1    | 1.5     |
| PM10 µg/m3  | 0x0012              | 0.1    | 1.8     |

\*Wrong value on PM10 Nb/L for firmware version 0x0853 and 0x854, corrected in the 0x0855 firmware.

**2.2.2.2           0x16 command reply (state)**

| Address | Cmd code | State (1 byte) | Checksum |
| ------- | -------- | -------------- | -------- |
| 0x81    | 0x16     | 0x04           | 0x65     |

\
The 0x16 command frame requests the State code. This reply is also the default response to other requests when the NextPM Lite has no data to send neither because the sensor has just been switched on nor because the sensor is in the Default State and/or Sleep State. See Status code description for State code decoding.

**2.2.2.3           0x17 command reply**

| Address | Cmd code | State | Firmware version | Checksum |
| ------- | -------- | ----- | ---------------- | -------- |
| 0x81    | 0x17     | 0x00  | 0x0034           | 0x34     |

\
The NextPM Lite replies to its firmware version. If the NextPM Lite is in the SLEEP mode, it will only reply to its state code.



#### 2.2.3 Writing command list <a href="#hx14gxiea98f" id="hx14gxiea98f"></a>

| Cmd code | Description                 | full command        |
| -------- | --------------------------- | ------------------- |
| 0x15     | Sleep mode entry / exit (1) | 0x81 0x15 0x6A      |
| 0x22     | Modbus address write (2)    | 0x81 0x22 0x03 0x5A |

(1) At each 0x15 command frame sent, the NextPM Lite changes its functional state alternately. To know its state (see Status code description) before sending the command frame, the user can send a 0x16 command frame

(2) The 0x22 command is used to write the new modbus address. The third byte (here 0x03) is the address, which must be different from 0. See 0x22 command reply.\


#### 2.2.4 Writing replies <a href="#hsvrmoaf0nin" id="hsvrmoaf0nin"></a>

**2.2.4.1          0x15 command reply (sleep mode)**

| address | Command code | State (1 byte) | Checksum |
| ------- | ------------ | -------------- | -------- |
| 0x81    | 0x15         | 0x01           | 0x69     |

\
When the NextPM Lite receives a sleep mode command frame, it turns into sleep mode, stops its functionalities (fan, laser and so, the PM measurements) and replies with a state code whose sleep flag is set.

During the sleep mode, if the NextPM Lite receives a new 0x15 command frame, it switches on and sends the first valid PM data after 15 seconds.

While the NextPM Lite is in sleep mode, it replies to all the other command frames by sending a 0x16 command code and the state code.

If the NextPM Lite is in the Default State and so in sleep mode, the 0x15 command frame turns on the sensor: it allows the user to try a restart without errors.

**2.2.4.2          0x22 command reply**

| address | Command code | State (1 byte) | Modbus address | Checksum |
| ------- | ------------ | -------------- | -------------- | -------- |
| 0x81    | 0x22         | 0x00           | 0x03           | 0x5A     |

\
The user can change the modbus device address using this command. The sensor replies to its new modbus address. The addresses start at 1 up to 15 (decimal values)

#### 2.2.5 Error codes <a href="#id-4d9oydrk93hf" id="id-4d9oydrk93hf"></a>

The NextPM Lite sensor replies the state code (see 0x16 command reply (state)) in these different cases:

* sleep mode
* default mode
* measurements not ready (in case of power on or sleep mode exit)

It won’t send back any response in case of:

* wrong checksum
* wrong command code
* wrong address
* command frame length too long (>5 bytes)

### 2.3 Modbus Protocol <a href="#id-2bn6wsx" id="id-2bn6wsx"></a>

The second way to get data from the NextPM Lite sensor is using the modbus protocol, respecting the modbus standard (v1.1b modbus specification compliance).\


#### 2.3.1 General information <a href="#fni7gn4wn53c" id="fni7gn4wn53c"></a>

#### 2.3.1.1          Bytes order <a href="#o0y26dilqa3v" id="o0y26dilqa3v"></a>

All registers are coded on 2 bytes (word) but some data can be coded on 2 or more registers.

If a data is coded on 4 bytes (2 words, so 2 registers), such as PM measurements, the second word is the most significant one.

Example: 0x01 0x03 0x04 0x4E 0x10 0x00 0x03 0xAC 0xDF, is the reply of PM2.5 measurement value request. The PM2.5 raw value is: (0x4E10) + (0x0003)\*65536 = 216 592. Then, a factor must be applied to get the measurement in the correct unit (see Readable registers list)

#### 2.3.1.2          Modbus default address <a href="#bn72zd1careh" id="bn72zd1careh"></a>

Default modbus address is 0x01 but it can be changed by sending a simple protocol write command frame or modbus command write frame. The change is saved into its memory and is kept even after a restart.\


2.3.1.3          Timings

Timings are the ones specified in Serial communication configuration:

* The NextPM Lite considers a timeout when there is 50ms of idle state between 2 bytes.
* The NextPM Lite delay is 50 ms (beginning of the reply to a request).\


#### 2.3.1.4          Function codes <a href="#id-6whfxgq676pt" id="id-6whfxgq676pt"></a>

NextPM Lite registers are accessible through 3 function codes:

* 0x03: read multiple holding registers
* 0x10: write multiple registers
* 0x17: Read/Write Multiple registers



#### 2.3.2 Readable registers list <a href="#id-2p2csry" id="id-2p2csry"></a>

_The following registers are only readable and accessible with a read or read/write multiple holding registers function command (0x03 or 0x17)_

| Register              | Name                                           | Description                                                                                |
| --------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 1 (0x01)              | Firmware version                               | Embedded firmware version                                                                  |
| 19 (0x13)             | Status                                         | NextPM Lite status (error codes)                                                           |
| 50-51 (0x32 - 0x33)   | PM1 10 sec average (Nb/L)                      | average over 10s of particles quantity per liter whose size is < 1μm (1)                   |
| 52-53 (0x34 - 0x35)   | PM2.5 10 sec average (Nb/L)                    | average over 10s of particles quantity per liter whose size is < 2.5μm (1)                 |
| 54-55 (0x36 - 0x37)   | PM10 10 sec average (Nb/L)                     | average over 10s of particles quantity per liter whose size is < 10μm (1)                  |
| 56-57 (0x38 - 0x39)   | PM1 10 sec average (µg/m3)                     | average over 10s of particles mass concentration (µg/m3) whose size is < 1μm (1) (2)       |
| 58-59 (0x3A - 0x3B)   | PM2.5 10 sec average (µg/m3)                   | average over 10s of particles mass concentration (µg/m3) whose size is < 2.5μm (1) (2)     |
| 60-61 (0x3C - 0x3D)   | PM10 10 sec average (µg/m3)                    | average over 10s of particles mass concentration (µg/m3) whose size is < 10μm (1) (2)      |
| 62-63 (0x3E - 0x3F)   | PM1 1 min average (Nb/L)                       | average over 1 min of particles quantity per liter whose size is < 1μm (1)                 |
| 64-65 (0x40 - 0x41)   | PM2.5 1 min average (Nb/L)                     | average over 1 min of particles quantity per liter whose size is < 2.5μm (1)               |
| 66-67 (0x42 - 0x43)   | PM10 1 min average (Nb/L)                      | average over 1 min of particles quantity per liter whose size is < 10μm (1)                |
| 68-69 (0x44 - 0x45)   | PM1 1 min average (µg/m3)                      | average over 1 min of particles mass concentration (µg/m3) whose size is < 1μm (1) (2)     |
| 70-71 (0x46 - 0x47)   | PM2.5 1 min average (µg/m3)                    | average over 1 min of particles mass concentration (µg/m3) whose size is < 2.5μm (1) (2)   |
| 72-73 (0x48 - 0x49)   | PM10 1 min average (µg/m3)                     | average over 1 min of particles mass concentration (µg/m3) whose size is < 10μm (1) (2)    |
| 74-75 (0x4A - 0x4B)   | PM1 10 min average (Nb/L)                      | average over 10 min of particles quantity per liter whose size is < 1μm (1)                |
| 76-77 (0x4C - 0x4D)   | PM2.5 10 min average (Nb/L)                    | average over 10 min of particles quantity per liter whose size is < 2.5μm (1)              |
| 78-79 (0x4E - 0x4F)   | PM10 10 min average (Nb/L)                     | average over 10 min of particles quantity per liter whose size is < 10μm (1)               |
| 80-81 (0x50 - 0x51)   | PM1 10 min average (µg/m3)                     | average over 10 min of particles mass concentration (µg/m3) whose size is < 1μm (1) (2)    |
| 82-83 (0x52 - 0x53)   | PM2.5 10 min average (µg/m3)                   | average over 10 min of particles mass concentration (µg/m3) whose size is < 2.5μm (1) (2)  |
| 84-85 (0x54 - 0x55)   | PM10 10 min average (µg/m3)                    | average over 10 min of particles mass concentration (µg/m3) whose size is < 10μm (1) (2)   |
| 100 (0x64)            | Fan command ratio (%)                          | Fan command ratio (%) (to be divided by 100)                                               |
| 101 (0x65)            | Laser status                                   | <p>10000 for laser on (to convert in hex)</p><p>0 for laser off</p>                        |
| 102 (0x66)            | Fan speed                                      | Fan rotation speed (Hz)                                                                    |
| 128-129 (0x80 - 0x81) | 0.3µm to 0.5µm quantity (Nb/L) 10 sec average  | average over 10s of particles quantity per liter whose size is between 0.3µm and 0.5µm (1) |
| 130-131 (0x82 - 0x83) | 0.5µm to 1.0µm quantity (Nb/L) 10 sec average  | average over 10s of particles quantity per liter whose size is between 0.5µm to 1.0µm (1)  |
| 132-133 (0x84 - 0x85) | 1.0µm to 2.5µm quantity (Nb/L) 10 sec average  | average over 10s of particles quantity per liter whose size is between 1.0µm to 2.5µm (1)  |
| 134-135 (0x86 - 0x87) | 2.5µm to 5.0µm quantity (Nb/L) 10 sec average  | average over 10s of particles quantity per liter whose size is between 2.5µm to 5.0µm (1)  |
| 136-137 (0x88 - 0x89) | 5.0µm to 10.0µm quantity (Nb/L) 10 sec average | average over 10s of particles quantity per liter whose size is between 5.0µm to 10.0µm (1) |

1. See example below explaining how to decode mass and quantity concentration.
2. Mass concentration should be divided by 1000 to get the value in µg/m3



**Measurement decoding example :**

* PM1, PM2.5 and PM10 concentrations request command: 01 03 00 32 00 24 E4 1E
* Sensor’s reply :01 03 48 62 4F 00 25 62 4F 00 25 62 4F 00 25 00 EC 00 00 00 EC 00 00 00 EC 00 00 6A 5D 00 13 99 6F 00 14 57 22 00 15 00 5E 00 00 01 82 00 00 03 A8 00 00 00 ED 00 17 CA FA 00 17 FE 29 00 17 00 A7 00 00 01 C8 00 00 02 69 00 00 77 09\

  * Calculation of the 10s average of the particles number concentration / liter whose size is < 1μm :
    * 0x624F is the less significant word and 0x0025 is the most significant word
    * the concatenate value (raw value) is 0x0025624F which is 2449999 in decimal. This is the measured physical value in Nb/L, averaged over 10 seconds.
  *   Calculation of the 10s average of the mass concentration of particles whose size is < 1μm :

      * 0x00EC is the less significant word and 0x0000 is the most significant word
      * the concatenate value (raw value) is 0x000000EC which is 236 in decimal
      * The measured physical value of PM1 is 0.236 µg/m3 , averaged over 10 seconds.



#### 2.3.3 Writable registers list <a href="#izkgalhf2a0a" id="izkgalhf2a0a"></a>

_The following registers are writable and accessible with a write or read/write multiple holding registers function command (0x10 or 0x17). These are advanced registers that need to be considered carefully before changed, as they may modify the sensor behaviour. We recommend contacting Tera Sensor before modifying them._

| Register   | Name                 | Description                                                       |
| ---------- | -------------------- | ----------------------------------------------------------------- |
| 88 (0x58)  | Modbus address       | <p>NextPM with Modbus protocol</p><p>( range is 1 to 15)</p>      |
| 100 (0x64) | Fan target ratio (%) | <p>Fan rotation setpoint (%)</p><p>(to be divided by 100) (1)</p> |
| 101 (0x65) | Laser function       | <p>0: turn off the laser</p><p>10000: turn on the laser</p>       |

1. _As it will change the behavior of the sensor and the reliability of the measurements, Tera Sensor does not recommend any change ._
2. _The baud rate of the simple and modbus communication can be changed by writing the identifier code corresponding to the desired baud rate value (see table below)._

### 2.4 I2C Protocol <a href="#wq4uaqsn4ee" id="wq4uaqsn4ee"></a>

#### 2.4.1 General information <a href="#h3u5vva1xj38" id="h3u5vva1xj38"></a>

#### 2.4.1.1          Bytes order <a href="#id-8cpns6wz3swz" id="id-8cpns6wz3swz"></a>

All registers are coded on 4 bytes (word).

Data coded on 4 bytes, such as PM measurements, the first word is the most significant one, and the second the less significant one, as follow:

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 115149.png" alt=""><figcaption></figcaption></figure>

#### 2.4.1.2          I2C default Address <a href="#gtfvt5myx7js" id="gtfvt5myx7js"></a>

The default I2C address is 0x04 and cannot be changed for now.

#### 2.4.1.3          Read and Write examples <a href="#id-59w9893r51xh" id="id-59w9893r51xh"></a>

Here is an example of a **read** communication:

* the master sends the following command: 0x04 0x01 0x05, to get firmware version:
  * 0x04 is the NextPM Lite send address command (0b0000 010**0**)
  * 0x01 is the firmware register code (see list below)
  * 0x05 is the NextPM Lite receive address command (0b0000 010**1**)
* the slave will reply: 0x1841 (for example)

Here is an example of a **write** communication:

* the master sends the following command: 0x04 0x1C 0x04 0x00 0x02, to change modbus: address.
  * 0x04 is the NextPM Lite send address command (0b0000 010**0**)
  * 0x1C is the firmware register code (see list below)
  * 0x04 is the NextPM Lite send address command (0b0000 010**0**)
  * 0x00 0x02 is the new value of the modbus address

#### 2.4.2 I2C read functions <a href="#mh7jz468s7py" id="mh7jz468s7py"></a>

| Register | Name | Description |
| -------- | ---- | ----------- |

| 00 | Status                                                                         | Error code or status of NextPM Lite                                                               |
| -- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| 01 | Firmware version                                                               | Embedded software version                                                                         |
| 06 | GetTenSecondesFastAverageResult.Mass\_1\_0\_L                                  | average over 10s of the mass concentration in µg/m3 of particles size < 1µm                       |
| 07 | GetTenSecondesFastAverageResult.Mass\_2\_5\_L                                  | average over 10s of the mass concentration in µg/m3 of particles size < 2.5µm                     |
| 08 | GetTenSecondesFastAverageResult.Mass\_10\_L                                    | average over 10s of the mass concentration in µg/m3 of particles size < 10µm                      |
| 09 | GetTenSecondesFastAverageResult.Nb\_1\_0\_L                                    | average over 10s of the Nb concentration in Pcs/L of particles size < 1µm                         |
| 10 | GetTenSecondesFastAverageResult.Nb\_2\_5\_L                                    | average over 10s of the Nb concentration in Pcs/L of particles size < 2.5µm                       |
| 11 | GetTenSecondesFastAverageResult.Nb\_10\_L                                      | average over 10s of the Nb concentration in Pcs/L of particles size < 10µm                        |
| 12 | 0.3µm to 0.5µm quantity (Nb/L) 10 sec average                                  | average over 10s of particles quantity per liter whose size is between 0.3µm and 0.5µm (1)        |
| 13 | 0.5µm to 1.0µm quantity (Nb/L) 10 sec average                                  | average over 10s of particles quantity per liter whose size is between 0.5µm to 1.0µm (1)         |
| 14 | 1.0µm to 2.5µm quantity (Nb/L) 10 sec average                                  | average over 10s of particles quantity per liter whose size is between 1.0µm to 2.5µm (1)         |
| 15 | 2.5µm to 5.0µm quantity (Nb/L) 10 sec average                                  | average over 10s of particles quantity per liter whose size is between 2.5µm to 5.0µm (1)         |
| 16 | 5.0µm to 10.0µm quantity (Nb/L) 10 sec average                                 | average over 10s of particles quantity per liter whose size is between 5.0µm to 10.0µm (1)        |
| 17 | GetOneMinuteAverageResult.Mass\_1\_0\_L                                        | average over 1 min of the mass concentration in µg/m3 of particles size < 1µm                     |
| 18 | GetOneMinuteAverageResult.Mass\_2\_5\_L                                        | average over 1 min of the mass concentration in µg/m3 of particles size < 2.5µm                   |
| 19 | GetOneMinuteAverageResult.Mass\_10\_L                                          | average over 1 min of the mass concentration in µg/m3 of particles size < 10µm                    |
| 20 | GetOneMinuteAverageResult.Nb\_1\_0\_L                                          | average over 1 min of the Nb concentration in Pcs/L of particles size < 1µm                       |
| 21 | GetOneMinuteAverageResult.Nb\_2\_5\_L                                          | average over 1 min of the Nb concentration in Pcs/L of particles size < 2.5µm                     |
| 22 | GetOneMinuteAverageResult.Nb\_10\_L                                            | average over 1 min of the Nb concentration in Pcs/L of particles size < 10µm                      |
| 23 | 0.3µm to 0.5µm quantity (Nb/L) 1 minute average                                | average over 1 minute of particles quantity per liter whose size is between 0.3µm and 0.5µm (1)   |
| 24 | 0.5µm to 1.0µm quantity (Nb/L) 1 minute average                                | average over 1 minute of particles quantity per liter whose size is between 0.5µm to 1.0µm (1)    |
| 25 | 1.0µm to 2.5µm quantity (Nb/L) 1 minute average                                | average over 1 minute of particles quantity per liter whose size is between 1.0µm to 2.5µm (1)    |
| 26 | 2.5µm to 5.0µm quantity (Nb/L) 1 minute average                                | average over 1 minute of particles quantity per liter whose size is between 2.5µm to 5.0µm (1)    |
| 27 | 5.0µm to 10.0µm quantity (Nb/L) 1 minute average                               | average over 1 minute of particles quantity per liter whose size is between 5.0µm to 10.0µm (1)   |
| 28 | GetTenMinutesAverageResult.Mass\_1\_0\_L                                       | average over 1 min of the mass concentration in µg/m3 of particles size < 1µm                     |
| 29 | GetTenMinutesAverageResult.Mass\_2\_5\_L                                       | average over 1 min of the mass concentration in µg/m3 of particles size < 2.5µm                   |
| 30 | GetTenMinutesAverageResult.Mass\_10\_L                                         | average over 1 min of the mass concentration in µg/m3 of particles size < 10µm                    |
| 31 | GetTenMinutesAverageResult.Nb\_1\_0\_L                                         | average over 1 min of the Nb concentration in Pcs/L of particles size < 1µm                       |
| 32 | GetTenMinutesAverageResult.Nb\_2\_5\_L                                         | average over 1 min of the Nb concentration in Pcs/L of particles size < 2.5µm                     |
| 33 | GetTenMinutesAverageResult.Nb\_10\_L                                           | average over 1 min of the Nb concentration in Pcs/L of particles size < 10µm                      |
| 35 | 0.3µm to 0.5µm quantity (Nb/L) 10 minutes average                              | average over 10 minutes of particles quantity per liter whose size is between 0.3µm and 0.5µm (1) |
| 36 | 0.5µm to 1.0µm quantity (Nb/L) 10 minutes average                              | average over 10 minutes of particles quantity per liter whose size is between 0.5µm to 1.0µm (1)  |
| 37 | 1.0µm to 2.5µm quantity (Nb/L) 10 minutes average                              | average over 10 minutes of particles quantity per liter whose size is between 1.0µm to 2.5µm (1)  |
| 38 | 2.5µm to 5.0µm quantity (Nb/L) 10 minutes average                              | average over 10 minutes of particles quantity per liter whose size is between 2.5µm to 5.0µm (1)  |
| 39 | 5.0µm to 10.0µm quantity (Nb/L) 10 minutes average                             | average over 10 minutes of particles quantity per liter whose size is between 5.0µm to 10.0µm (1) |
| 41 | Fan speed                                                                      | Fan rotation speed (Hz)                                                                           |
| 42 | Laser status                                                                   | <p>10000 for laser on</p><p>0 for laser off</p>                                                   |
| 56 | PM1 + PM2.5 + PM10 (µg/m3) averaged over 10 seconds and updated every 1 second | 3 Concentrations (µg/m3) averaged over 10 seconds and updated every 1 second                      |
| 57 | PM1 + PM2.5 + PM10 (µg/m3) averaged over 10 min and updated every 1 minute     | 3 Concentrations (µg/m3) averaged over 10 min and updated every 1 minute                          |
| 58 | PM1 + PM2.5 + PM10 (µg/m3) averaged over 1 min and updated every 10 seconds    | 3 Concentrations (µg/m3) averaged over 1 min and updated every 10 seconds                         |
| 59 | PN1 + PN2.5 + PN10 (Nb/L) averaged over 10 seconds and updated every 1 second  | 3 Concentrations (Nb/L) averaged over 10 seconds and updated every 1 second                       |
| 60 | PN1 + PN2.5 + PN10 (Nb/L) averaged over 10 min and updated every 1 minute      | 3 Concentrations (Nb/L) averaged over 10 min and updated every 1 minute                           |
| 61 | PN1 + PN2.5 + PN10 (Nb/L) averaged over 1 min and updated every 10 seconds     | 3 Concentrations (Nb/L) averaged over 1 min and updated every 10 seconds                          |
| 62 | 5 Channels (Nb/L) averaged over 10 seconds and updated every 1 second          | Concentrations (Nb/L) averaged over 10 seconds and updated every 1 second                         |
| 63 | 5 Channels (Nb/L) averaged over 10 min and updated every 1 minute              | Concentrations (Nb/L) averaged over 10 min and updated every 1 minute                             |
| 64 | 5 Channels (Nb/L) averaged over 1 min and updated every 10 seconds             | Concentrations (Nb/L) averaged over 1 min and updated every 10 seconds                            |





***



## 3. Test software <a href="#id-3o7alnk" id="id-3o7alnk"></a>

Tera Sensor provides software that can help the user to communicate with the sensor and test it before working on the integration.

![](<../../.gitbook/assets/4 (1).png>)

Note: The computer running the software should have Windows 10 or newer version.

The only protocol available with this software is Modbus.





***



## 4. NextPM Lite Mechanical tips <a href="#id-30j0zll" id="id-30j0zll"></a>

In progress





***



## 5. Sequential measurements <a href="#frmoh9pmzfq5" id="frmoh9pmzfq5"></a>

The NextPM Lite sensor is a good option for sequential measurements, but there are conditions to respect in order to obtain the best results possible.

The NextPM Lite sensor is equipped with a sleep mode to reduce power consumption, or it is also possible to add a power switch to turn the sensor completely off.

After an off/sleep period, the sensor is turned back on, it is needed to wait 15 seconds before the sensor is fully operational (mainly for the fluidic path to become stable after the fan restart) . It is also recommended to wait an additional 20 seconds minimum (1 minute is better) before collecting the data from the 1 minute average command for PM1, PM2.5 and PM10.

In this case the results obtained will be reliable and can be compared with other instruments if needed.

Table resuming the example cited above:

<figure><img src="../../.gitbook/assets/Capture d&#x27;écran 2025-03-22 115837.png" alt=""><figcaption></figcaption></figure>





***





## 6. Electrical integrity <a href="#a87ppbkc2s87" id="a87ppbkc2s87"></a>

As the EMC shield of the sensor is connected to its ground, the integrator should avoid a mechanical contact to the system earth / ground, by using a non conductive material or an electrical insulation.

This will avoid any unwanted current path flow in case of power supply isolation fault, which would damage the sensor





***



## Revision history <a href="#id-6jw5dw9t2ha9" id="id-6jw5dw9t2ha9"></a>

| Date       | Revision | Changes                                                                                                                                                                                                                                                                                                                                                         |
| ---------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2024/11/05 | 0.1      | <p>First version of NextPM Lite Userguide</p><p>Includes :</p><ul><li>general information</li><li>Communication of simplified, modbus and I2C protocols</li><li>Test software description</li><li>Sequential use description</li><li>Electrical integrity</li></ul><p>Does not include mechanical tips</p><p>Applied for firmware version 0x0853 and 0x0854</p> |

Please visit [https://tera-sensor.com/fr/](https://tera-sensor.com/fr/) to find more information and to contact the sales services
