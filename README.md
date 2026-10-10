# 8051-Based Air Quality Monitoring System Using ADC0808

A sensor-based air-quality monitoring prototype built around an AT89C51-compatible 8051 microcontroller and an ADC0808 analog-to-digital converter. The system reads two analog sensor channels, displays the raw readings on a 16×2 LCD, and provides visual and audible alerts through three LEDs and a buzzer based on programmed threshold logic.

> The system reports raw ADC counts from its sensor inputs. It does not calculate certified or calibrated AQI values.

---

## Table of Contents

1. [Project Summary](#project-summary)
2. [Project Overview](#project-overview)
3. [Hardware Components](#hardware-components)
4. [Software and Tools](#software-and-tools)
5. [Hardware Connections](#hardware-connections)
6. [Program Structure](#program-structure)
7. [Working Principle](#working-principle)
8. [LCD Output](#lcd-output)
9. [LED and Buzzer Logic](#led-and-buzzer-logic)
10. [Build and Simulation](#build-and-simulation)
11. [Testing and Results](#testing-and-results)
12. [Learning Outcomes](#learning-outcomes)
13. [Limitations](#limitations)
14. [Future Improvements](#future-improvements)
15. [Repository Structure](#repository-structure)
16. [License](#license)

---

## Project Summary

| Item | Description |
|---|---|
| Project type | Embedded systems prototype |
| Microcontroller | AT89C51-compatible 8051 |
| Data acquisition | ADC0808 (8-bit, 8-channel multiplexed) |
| Inputs | Two analog sensor channels: [sensor models] |
| Outputs | 16×2 LCD (4-bit mode), red, yellow, and green LEDs, buzzer |
| Clock | 11.0592 MHz crystal |
| Language | Embedded C |
| Tools | Keil µVision, Proteus, Git and GitHub |
| Core concepts | ADC interfacing, LCD interfacing, timer-based delays, conditional logic, peripheral control |

---

## Project Overview

The project is an air-quality monitoring prototype using an 8051-family microcontroller, an ADC0808, two analog sensor inputs, and a 16×2 LCD.

The ADC0808 converts the analog sensor signals into 8-bit digital values. The microcontroller reads the two channels one after the other and displays both readings on the LCD. Three LEDs (green, yellow, and red) indicate different conditions, and a buzzer gives an audible warning when the programmed danger condition is met.

The project demonstrates the following embedded systems concepts:

- ADC interfacing and channel selection
- LCD interfacing in 4-bit mode
- Embedded C programming with modular functions
- Conditional logic for threshold-based outputs
- Timer-based delays using Timer 0
- Peripheral control (LEDs and buzzer)

The displayed values are raw ADC counts (0–255). They are not calibrated concentrations and are not AQI values.

---

## Hardware Components

| Component | Details |
|---|---|
| AT89C51-compatible 8051 microcontroller | Main controller |
| ADC0808 | 8-bit ADC with multiplexed analog inputs |
| Analog sensor inputs (2) | [Sensor models] |
| 16×2 LCD | Operated in 4-bit mode |
| Green LED | [Pin] |
| Yellow LED | P2.1 |
| Red LED | P2.0 |
| Buzzer | P3.7 |
| Crystal oscillator | 11.0592 MHz |
| Resistors, wiring, power supply, supporting components | [Values and ratings] |

---

## Software and Tools

| Tool | Purpose |
|---|---|
| Embedded C | Firmware development |
| Keil µVision | Code development and compilation |
| Proteus | Circuit simulation |
| Git and GitHub | Source-code management and documentation |

---

## Hardware Connections

### ADC0808 to 8051

| ADC0808 Signal | 8051 Pin |
|---|---|
| D0–D7 | P1.0–P1.7 |
| Address A | P3.0 |
| Address B | P3.1 |
| Address C | P3.2 |
| ALE | P3.3 |
| START | P3.4 |
| EOC | P3.5 |
| OE | P3.6 |

The ADC0808 clock input, reference voltage (VREF) connections, and supply connections are: [clock source], [VREF connection], [supply connection].

### 16×2 LCD to 8051

The LCD uses 4-bit communication, so only the upper four data lines are connected.

| LCD Signal | 8051 Pin |
|---|---|
| RS | P2.2 |
| Enable | P2.3 |
| D4 | P2.4 |
| D5 | P2.5 |
| D6 | P2.6 |
| D7 | P2.7 |
| RW | Ground (write-only operation) |

### LEDs and Buzzer

| Component | 8051 Pin |
|---|---|
| Red LED | P2.0 |
| Yellow LED | P2.1 |
| Green LED | [Pin] |
| Buzzer | P3.7 |

Because P3.6 is used by the ADC0808 OE signal, the green LED is connected to [Pin] instead.

---

## Program Structure

The firmware is divided into separate functions to make it easier to read, test, and debug.

| Function | Description |
|---|---|
| `main()` | Initializes the LCD and ADC control signals, reads both sensor channels, updates the LCD, and calls the LED/buzzer control function. |
| `ADC_Channel1()` | Selects ADC channel 1, starts a conversion, waits for completion, and returns the digital reading. |
| `ADC_Channel0()` | Selects ADC channel 0 and returns the digital reading. |
| `lcd_init()` | Initializes the LCD in 4-bit mode. |
| `lcd_nibble()` | Sends four bits (one nibble) to the LCD. |
| `lcd_cmd()` | Sends a command to the LCD, such as clear display or set cursor position. |
| `lcd_data()` | Sends a single character to the LCD. |
| `lcd_string()` | Displays a string by sending its characters one at a time. |
| `lcd_num()` | Converts a number into three display digits. |
| `MSDelay()` | Generates timer-based delays using Timer 0. |
| `delay()` | Generates a simple software delay. |
| `led_buzzer()` | Controls the LEDs and buzzer using conditional statements based on both sensor readings. |

**Notes:**

- The LCD receives each byte as two 4-bit nibbles (high nibble, then low nibble), which is why `lcd_nibble()` is a separate helper.
- `lcd_num()` splits a value from 0 to 255 into hundreds, tens, and ones so each digit can be displayed as a character.
- `MSDelay()` uses Timer 0, so its timing depends on the 11.0592 MHz crystal. `delay()` is a software loop and is less precise.

---

## Working Principle

1. The analog sensors produce electrical signals.
2. The ADC0808 selects an input channel and converts the analog signal into an 8-bit digital value.
3. The 8051 reads the digital result through Port 1.
4. The program reads channel 1 and channel 0 in sequence.
5. The two sensor readings are displayed on the LCD.
6. The `led_buzzer()` function evaluates the readings against the programmed conditions.
7. The matching LED is activated, and the buzzer sounds when the configured danger condition is met.

### Block Diagram

```text
Analog Sensor Inputs → ADC0808 → 8051 Microcontroller → LCD Display + LEDs + Buzzer
```

### ADC0808 Control Signals

| Signal | Role |
|---|---|
| A, B, C | Address lines that select one of the multiplexed input channels |
| ALE | Address Latch Enable: latches the selected channel address |
| START | Starts the analog-to-digital conversion |
| EOC | End of Conversion: indicates when the result is ready |
| OE | Output Enable: places the converted 8-bit result on the data lines (D0–D7) |

---

## LCD Output

The LCD shows the two channel readings as raw ADC counts:

```text
S1:125
S2:087
```

The values above are sample readings used to illustrate the display format.

---

## LED and Buzzer Logic

The `led_buzzer()` function compares the sensor readings against the programmed thresholds and activates the matching output.

| Condition | Green LED | Yellow LED | Red LED | Buzzer |
|---|---|---|---|---|
| [Condition 1] | [State] | [State] | [State] | [State] |
| [Condition 2] | [State] | [State] | [State] | [State] |
| [Condition 3] | [State] | [State] | [State] | [State] |

The thresholds are set at approximately 100, 150, and 200 ADC counts. These are development values chosen for demonstration. They are not official AQI breakpoints, and they were selected for the sensors in this prototype rather than validated against a reference instrument.

---

## Build and Simulation

### Build the Firmware (Keil µVision)

1. Create a new project in Keil µVision and select the AT89C51 (or compatible) target device.
2. Add `Code/main.c` to the project.
3. In the target options, set the crystal frequency to 11.0592 MHz.
4. Enable "Create HEX File" in the output options and build the project.

### Simulate the Circuit (Proteus)

1. Open the Proteus project from the `Proteus/` folder.
2. Load the generated HEX file into the microcontroller component.
3. Set the microcontroller clock frequency to 11.0592 MHz.
4. Provide a clock signal to the ADC0808 and set its reference voltage: [clock source and VREF value].
5. Connect the sensor inputs, or use adjustable analog sources in place of the sensors: [simulation source].
6. Run the simulation and observe the LCD, LEDs, and buzzer.

---

## Testing and Results

Test results are recorded below only where they have been verified.

| # | Test | Result |
|---|---|---|
| 1 | ADC channel selection and conversion | [Verified result] |
| 2 | Reading both channels | [Verified result] |
| 3 | LCD output | [Verified result] |
| 4 | Individual LED operation | [Verified result] |
| 5 | Buzzer operation | [Verified result] |
| 6 | Combined LED and buzzer logic | [Verified result] |
| 7 | Response to different sensor inputs | [Verified result] |

### Screenshots



![Proteus circuit diagram](Images/circuit.png)





![LCD output](Images/output.png)





![System block diagram](Images/block_diagram.png)



---

## Learning Outcomes

- **ADC interfacing with an 8051:** connecting an 8-bit parallel ADC to a microcontroller port and reading its result.
- **Channel selection:** using address lines A, B, and C to choose a multiplexer input.
- **ALE, START, EOC, and OE:** understanding the timing sequence between latching the address, starting conversion, waiting for completion, and reading the output.
- **LCD in 4-bit mode:** sending commands and data as two nibbles and following the initialization sequence.
- **C functions and conditional logic:** structuring code into reusable functions and applying threshold-based decisions.
- **Timer-based delays:** using Timer 0 for accurate timing instead of relying only on software loops.
- **Pin planning and conflict resolution:** recognizing that P3.6 is already used by ADC OE and assigning the green LED to a different pin.
- **System integration:** combining sensor acquisition, display, and output control in one firmware program.

---

## Limitations

- The ADC channels are read sequentially rather than simultaneously.
- Raw ADC values are not equivalent to calibrated AQI values.
- Accuracy depends on sensor calibration and environmental conditions.
- The thresholds are development values and have not been validated against a reference instrument.

---

## Future Improvements

The following are proposed extensions and are not implemented in this version:

- Sensor calibration using reference measurements
- Filtering or averaging of sensor readings
- A validated air-quality classification algorithm
- Wireless monitoring using an ESP8266 or ESP32
- Remote alerts and data logging
- Improved fault detection and system reliability

---

## Repository Structure

```text
8051-AQI-Monitoring-System/
├── README.md
├── Code/
│   └── main.c
├── Proteus/
│   └── simulation files
└── Images/
    ├── block_diagram.png
    ├── circuit.png
    └── output.png
```

| Path | Contents |
|---|---|
| `README.md` | Project documentation |
| `Code/main.c` | Embedded C firmware |
| `Proteus/` | Proteus simulation project files |
| `Images/block_diagram.png` | System block diagram |
| `Images/circuit.png` | Proteus circuit diagram |
| `Images/output.png` | LCD output screenshot |

---

## License

[Choose a license, such as MIT or GPL-3.0, and add it here.]
