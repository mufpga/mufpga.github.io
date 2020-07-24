# MicroFPGA

MicroFPGA is an electronic platform generating signals that are commonly needed when controlling a microscope. It is based on the Au and Cu FPGA (Alchitry).

The available types of signals are:

| Name            | I/O    | Function                                                     | Default number |
| --------------- | ------ | :----------------------------------------------------------- | :------------: |
| Laser trigger   | Output | These digital signals require that a camera trigger signal be given as an input to the FPGA. The laser triggers are then individually generated following three parameters (mode, duration and sequence) in order to control lasers digital input. |       10       |
| TTL             | Output | TTL signals are either HIGH or LOW, and can be used to control devices such as flip-mirrors or switches. |       10       |
| Servo           | Output | The servos signals are periodic digital signals (20 ms period and 1 to 2 ms pulses) used commonly to move servomotors. The MicroFPGA servo signals are turned off after 10 ms in order to prevent vibrations. |       10       |
| PWM             | Output | PWM signals are similar to the Servo signals, except they are not switched off after 10 s. Together with a low-pass electronic filter, they can be used to produce analog signals, used for instance with AOTF. |       10       |
| Analog read-out | Input  | Analog inputs are limited to 0-1 V and are typically used to measure temperature, laser position or any measurement signal. |   8 (fixed)    |



**GENERAL FIGURE here**



### Laser trigger

Cameras (whether EMCCD or sCMOS) are usually delivered with an output cable to trigger devices. The trigger output is a TTL signal (high/low) that notifies the devices when the camera is registering the pixel values or when it is exposing. Connecting directly the lasers trigger input (when it is available) to the camera trigger output has the advantage to prevent unnecessary exposure of the sample to laser light: the laser is only emitting when the camera is exposing. 

However, direct triggering is not flexible and does not allow for pulsing the lasers to reduce effective laser power or alternating the triggering between two lasers. MicroFPGA act as a camera signal modulator to produce a laser trigger that can be pulsed or synchronized between lasers. The camera signal must be provided to the FPGA (refer to the [pins mapping](pins_br.md)).

:warning: **Do not supply the camera signal with more than 3.3V to the FPGA** :warning:

Often EMCCD have a 5 V trigger, it then needs to be scaled down to 3.3 V. sCMOS cameras usually run on 3.3 V.

MicroFPGA offers several trigger mode:

| Trigger mode |         Parameters         |                         Description                          |
| :----------: | :------------------------: | :----------------------------------------------------------: |
|     OFF      |                            |                   The laser is always off                    |
|      ON      |                            |                    The laser is always on                    |
|    RISING    | **duration<br />sequence** | The laser is pulsed for **duration** micro-seconds, on each **rising** edge |
|   FALLING    | **duration<br />sequence** | The laser is pulsed for **duration** micro-seconds, on each **falling** edge |
|    CAMERA    |        **sequence**        |           The laser is follows the camera trigger            |

You need to only provide a single camera input. In addition to **mode** and **duration**, a third parameter can be set: the **sequence**. The **sequence** corresponds to a 16 bits number which encode a sequence of trigger. MicroFPGA reads the binary number from the most-significant bit to the least significant one. If the bit is 1, then the laser will be triggered during the next camera exposure. If it is 0, it will not be triggered. The sequence is applied in RISING, FALLING and CAMERA modes. The lasers are synchronized, allowing alternating sequences for instance (see examples).  



**LASER TRIGGER FIGURE HERE**



**Examples**:


|                 Decimal                  |                 Binary                 |                     Description                     |
| :--------------------------------------: | :------------------------------------: | :-------------------------------------------------: |
|                    0                     |            0000000000000000            |               The laser is always off               |
|                  65535                   |            1111111111111111            |           The laser is on at every frames           |
|                  43690                   |            1010101010101010            | The laser is on every two frames (starting with on) |
|                  51884                   |            1100101010101100            |                     See pattern                     |
| **Laser1**: 43690 <br> **Laser2**: 21845 | 1010101010101010 <br> 0101010101010101 |      **Laser1** and **Laser2** are alternating      |

Use a [binary-to-decimal converter](https://www.binaryhexconverter.com/binary-to-decimal-converter "One binary to decimal converter") to get the sequence right.

**Summary**, for each laser (laser number = #):

| Property  |                         Description                          |                             Use                              |
| :-------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   Mode#   | Triggering mode of the laser <br>(OFF\ON\RISING\FALLING\CAMERA) | OFF = 0 <br> ON = 1 <br> RISING = 2 <br> FALLING = 3 <br> CAMERA = 4 |
| Duration# | Pulse length of the trigger in us <br>(RISING\FALLING modes) |                          0-65535 us                          |
| Sequence# | Decimal value corresponding to a 16 bits <br>binary sequence of ON and OFF <br>(RISING\FALLING\CAMERA modes) |                           0-65535                            |

### TTL

TTL signals are often used to set a device on/off or in/out. A typical use are electronic flip-mirrors, in which a TTL low will send the mirror to its _out_ position, and a TTL high to its _in_ position.

| Property | Value |
| :------: | :---: |
|    ON    |   1   |
|   OFF    |   0   |

### Servo

Servos typically require three signals: power, ground and position. We advise supplying the power with another device than the FPGA to avoid drawing too little current.

A [servo position signal](https://en.wikipedia.org/wiki/Servo_control "Wikipedia Servo control") is usually a PWM signal between 1ms and 2 ms, with a period of 20 ms. MicroFPGA follows this standard with a resolution on the pulse width of 16 bits:

| Position | Value |
| :------: | :---: |
| Minimum  |   0   |
| Maximum  | 65535 |

Additionally, MicroFPGA turns off the servo signal after 10 seconds in order to avoid vibrations on the optical table.

### PWM

PWMs are similar to the servo signals, but are not switched off. MicroFPGA uses 8 bits values (0-255) to set the duty cycle value and a period of 1.3 ms. PWMs are particularly interesting as they can be used to create an analog signal using a low-pass filter.

### Analog read-out

Analog inputs are only available with the Au FPGA.

:warning:  **Do not supply inputs with more than 1.8V to the AU FPGA analog inputs​ :warning: **

MicroFPGA has eight analog read-out pins. Using the Au FPGA, you can read-out the measured voltage scaled to 0-65535:

| Reading | Voltage |
| :-----: | :-----: |
|    0    |   0 V   |
|  65535  |   1 V   |