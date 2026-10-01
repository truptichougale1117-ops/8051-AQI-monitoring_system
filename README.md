# 8051-AQI-monitoring_system

## Project Overview

This project is a microcontroller-based air quality monitoring system
using the 8051 microcontroller and ADC0808.

The system is designed to acquire analog signals from two gas/air-quality
sensors. Since the 8051 does not have a built-in ADC, the ADC0808 is used
to convert the analog sensor signals into 8-bit digital data.

The 8051 selects the required ADC channel, starts the conversion,
waits for the conversion to complete, and reads the digital output
through Port 1. 


## Current Progress
- 8051 microcontroller setup completed
- ADC0808 interfacing with 8051 completed
- ADC channel selection implemented
- ADC conversion and EOC monitoring implemented
- 8-bit ADC data reading through Port 1 implemented
- Two sensor channels being interfaced
- Proteus simulation developed for ADC interfacing

## Hardware Used

- AT89C51 (8051 Microcontroller)
- ADC0808 Analog-to-Digital Converter
- Sensor 1 (MQ-135)
- Sensor 2 (DUST SENSOR)
- 11.0592 MHz crystal oscillator
- 640 kHz clock for ADC0808
- +5 V power supply

## Why ADC0808 is Required

The sensors used in this project provide an analog voltage signal.

The AT89C51 (8051) does not have a built-in Analog-to-Digital Converter (ADC). Therefore, an external ADC is required to convert the analog sensor signal into digital data.

The ADC0808 is used for this purpose. It converts the analog input into an 8-bit digital value, which can then be read and processed by the 8051 through Port 1.

### Basic Data Flow

Sensor
↓
Analog Voltage
↓
ADC0808
↓
8-bit Digital Data
↓
8051 Microcontroller




## ADC0808–8051 Interfacing

The ADC0808 is interfaced with the AT89C51 using Port 1 for the 8-bit
digital data and selected pins of Port 3 for channel selection and
control signals.

| ADC0808 Signal | 8051 Pin | Function |
|---|---|---|
| D0–D7 | P1.0–P1.7 | 8-bit ADC data |
| A | P3.0 | Channel selection |
| B | P3.1 | Channel selection |
| C | P3.2 | Channel selection |
| ALE | P3.3 | Address latch enable |
| START | P3.4 | Starts ADC conversion |
| EOC | P3.5 | Indicates end of conversion |
| OE | P3.6 | Enables ADC output |
| CLOCK | External 640 kHz clock | ADC clock |



## ADC Channel Selection

The ADC0808 has multiple analog input channels. The 8051 selects the
required channel using the A, B and C address lines.

In this project, two ADC channels are used for the two sensor inputs.

| C | B | A | Selected Channel |
|---|---|---|---|
| 0 | 0 | 0 | Channel 0 |
| 0 | 0 | 1 | Channel 1 |

### Channel 0

For Channel 0:

A = 0  
B = 0  
C = 0

The 8051 sets P3.0, P3.1 and P3.2 accordingly.

### Channel 1

For Channel 1:

A = 1  
B = 0  
C = 0

The 8051 sets P3.0 = 1, while P3.1 and P3.2 remain 0.

The 8051 reads the two channels one after another rather than reading
both analog inputs simultaneously.



## ADC Conversion Process

The ADC0808 conversion process is controlled by the 8051 using the
channel-select lines, ALE, START, EOC and OE signals.

### Conversion Sequence

The 8051 performs the following sequence for each sensor reading:

1. **Channel Selection**

   The required ADC channel is selected using the A, B and C address
   lines.

2. **Address Latching**

   ALE (Address Latch Enable) is made HIGH and then LOW to latch the
   selected channel address inside the ADC0808.

3. **Start Conversion**

   A START pulse is applied to the ADC0808 to initiate the
   analog-to-digital conversion.

4. **Monitor EOC**

   The 8051 monitors the EOC (End of Conversion) signal and waits for
   the ADC conversion to complete.

5. **Enable Output**

   After conversion is completed, OE (Output Enable) is made HIGH.

6. **Read ADC Data**

   The 8-bit converted data available at D0-D7 is read through Port 1
   of the 8051.

7. **Disable Output**

   OE is then made LOW before starting the next conversion.

### Conversion Flow

Sensor
   ↓
Analog Input
   ↓
Channel Selection
   ↓
ALE
   ↓
START Pulse
   ↓
ADC0808 Conversion
   ↓
EOC Monitoring
   ↓
OE = HIGH
   ↓
Read D0-D7
   ↓
8051 Port 1
   ↓
Digital Sensor Value


## Debugging and Problems Encountered

During the Proteus simulation, several timing and control-signal
issues were observed and debugged.

### 1. EOC Initial-State Problem

Initially, the EOC signal was not initialized correctly in the
8051 program.

The program was updated to explicitly set:

EOC = 1;

This was required for the software polling sequence used in the
program, where the controller first waits for EOC to change from its
initial HIGH state and then waits for the conversion to complete.

Without the expected initial EOC state, the program could remain
waiting in the EOC polling loop and the ADC conversion sequence did
not proceed as expected in the simulation.

### 2. START Pulse and Delay Problem

During debugging, a relatively long software delay was initially
used between START HIGH and START LOW.

For example:

SC = 1;
MSDelay(1);
SC = 0;

This created an approximately millisecond-scale START pulse, which
was unnecessarily long for the ADC conversion control sequence.

Since the ADC was operated with a 640 kHz clock, the START signal was
changed to a short pulse using NOP instructions:

SC = 1;
_nop_();
_nop_();
_nop_();
SC = 0;

This allowed the START signal to be controlled at the required
short time scale.

### 3. EOC and START Signal Debugging Observation

During early Proteus debugging, an incorrect connection/configuration
was also tested in which the START and EOC signals were connected
together.

This connection is **not part of the intended ADC0808 hardware
interface and should not be used in the final circuit**.

However, during simulation it produced signal transitions that
appeared to allow the conversion sequence to progress. This helped
identify the relationship between the START and EOC signals and led
to correction of the control-signal implementation.

The final design keeps START and EOC as **separate signals**:

START → controlled by the 8051  
EOC   → generated by the ADC0808 and monitored by the 8051

### 4. Final Debugging Result

After correcting the initial EOC condition, reducing the START pulse
duration, and keeping START and EOC as separate control signals, the
ADC0808 conversion sequence could be observed correctly in the
Proteus simulation.

The debugging process helped verify the importance of:

- Correct ADC control-signal initialization
- Correct START pulse timing
- Proper EOC monitoring
- Correct channel selection
- Separate START and EOC connections
- Correct ADC clock operation



## Software Implementation

The ADC0808 interface is programmed using Embedded C in Keil µVision.

The program uses the following control signals:

- A, B, C → Select the ADC input channel
- ALE → Latches the selected channel
- START → Initiates ADC conversion
- EOC → Monitors the conversion status
- OE → Enables the ADC data output
- P1 → Reads the 8-bit converted data

### Program Sequence

The program continuously performs the following operations:

1. Select Channel 1.
2. Latch the channel address using ALE.
3. Generate a short START pulse.
4. Monitor EOC until conversion is completed.
5. Enable OE.
6. Read the 8-bit ADC value through Port 1.
7. Disable OE.
8. Select Channel 0.
9. Repeat the same conversion process.
10. Continue this process continuously.



## Proteus Simulation

The ADC0808 and AT89C51 interfacing was tested using Proteus simulation.

The simulation was used to verify:

- ADC channel selection
- ALE control
- START pulse generation
- EOC response
- OE control
- 8-bit ADC data output
- Reading ADC data through Port 1
- Sequential reading of two sensor channels
- ADC operation with a 640 kHz clock



### Simulation Setup

The AT89C51 controls the ADC0808 through Port 3, while Port 1 is used
to receive the 8-bit ADC output.

The ADC0808 is operated using an external 640 kHz clock.

Two analog inputs are connected to two ADC channels. The 8051 selects
one channel at a time and reads the corresponding digital value.

### Simulation Result

The Proteus simulation is currently used to verify the ADC interfacing
and conversion sequence before connecting the actual sensors and
hardware.

> Note: The current stage verifies ADC interfacing and data acquisition.
> Complete AQI calculation and final sensor calibration will be added
> in later stages of the project.

### Main Code Structure

```c
while(1)
{
    /* Select ADC channel */

    /* Latch address */
    ALE = 1;
    ...
    ALE = 0;

    /* Start conversion */
    SC = 1;
    _nop_();
    _nop_();
    _nop_();
    SC = 0;

    /* Wait for conversion */
    while(EOC == 1);
    while(EOC == 0);

    /* Read ADC result */
    OE = 1;
    value = MYDATA;
    OE = 0;
}


