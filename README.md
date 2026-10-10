# 8051-Based Air Quality Monitoring System Using ADC0808

## Overview

This embedded-systems project uses an 8051-family microcontroller and an ADC0808 analog-to-digital converter to read two analog sensor inputs. The firmware displays the readings on a 16×2 LCD and controls LEDs and a buzzer according to programmed thresholds.

The project was initially developed during my second year and is now being documented on GitHub.

> 

## Project Status

- **Proteus simulation:** Verified during project work.
- **Physical hardware:** Assembled during the original project, but its operation has not been re-verified recently.
- **Sensor thresholds:** Experimental values selected during earlier development. The original numerical clean-air readings are no longer available.
- **Calibration and validation:** The thresholds have not been validated against a suitable reference instrument.
- **Next verification steps:** Confirm the final pin assignments, repeat sensor measurements, and test the physical circuit.

## Features

- Reads two analog inputs through separate ADC0808 channels.
- Displays both readings on a 16×2 LCD using 4-bit mode.
- Organizes the firmware into functions for ADC acquisition, LCD operations, numerical display, and output control.
- Controls LEDs and a buzzer using programmed threshold conditions.
- Demonstrates 8051 peripheral interfacing, timer-based delays, and conditional logic.

The listed behavior should be considered implemented only to the extent confirmed by the current source code and simulation.

## Hardware Components

- 8051-family microcontroller — confirm the exact part number used in the final circuit.
- ADC0808 8-bit analog-to-digital converter.
- Two analog sensor modules — MQ135 and MQ7.
- 16×2 LCD.
- Red, yellow, and green LEDs.
- Buzzer.
- 11.0592 MHz crystal.
- Supporting components, wiring, reset circuit, and power supply.

## Software and Tools

- **Embedded C** — firmware.
- **Keil µVision** — code development and compilation.
- **Proteus** — circuit simulation.
- **Git and GitHub** — source control and project documentation.

## How It Works

1. The sensor modules provide analog voltage signals to the ADC0808 input channels.
2. The microcontroller selects an ADC channel using address lines A, B, and C.
3. The firmware controls the ADC conversion sequence and waits for the End of Conversion (EOC) signal.
4. The ADC's 8-bit digital output is enabled and read through Port 1.
5. The firmware reads the two channels sequentially.
6. The readings are displayed on the 16×2 LCD.
7. The `led_buzzer()` function evaluates the readings against the programmed conditions and controls the indicators.

The ADC0808 converts one selected channel at a time, so the two readings are acquired sequentially rather than simultaneously.

```text
Analog sensor inputs → ADC0808 → 8051 microcontroller → LCD
                                                    └→ LEDs and buzzer
```

## Hardware Connections

The following mapping is based on the project notes. Confirm it against the actual circuit and source code before using it for physical wiring.

### ADC0808 to 8051

| ADC0808 signal | 8051 connection |
|---|---|
| D0–D7 | P1.0–P1.7 |
| Address A | P3.0 |
| Address B | P3.1 |
| Address C | P3.2 |
| ALE | P3.3 |
| START / SC | P3.4 |
| EOC | P3.5 |
| OE | P3.6 |

* **Power supply:** The circuit is powered using a DC supply from a voltage-regulator circuit designed for the project. The ADC0808 supply and ground connections are connected to the appropriate power rails.
* **ADC clock in simulation:** A clock source was used in Proteus to provide the ADC0808 clock signal.
* **ADC clock in hardware:** The ADC clock is generated using a resistor-capacitor (RC) clock circuit.
* **Reference voltage:** The ADC0808 reference-voltage connections should be documented according to the actual circuit wiring.
 

### 16×2 LCD to 8051

The LCD is configured for 4-bit communication.

| LCD signal | 8051 connection |
|---|---|
| RS | P2.2 |
| Enable (E) | P2.3 |
| D4 | P2.4 |
| D5 | P2.5 |
| D6 | P2.6 |
| D7 | P2.7 |
| RW | Ground (write-only operation) |

### LEDs and Buzzer

| Output | Pin noted in the project |
|---|---|
| Red LED | P2.0 |
| Yellow LED | P2.1 |
| Green LED | P3.6 |
| Buzzer | P3.7 |

## Software Organization

The firmware is written in Embedded C. Confirm the function names and descriptions against the source file included in this repository.

| Function | Purpose |
|---|---|
| `main()` | Initializes the required pins and peripherals, reads both channels, updates the LCD, and calls the output-control function. |
| `ADC_Channel0()` and `ADC_Channel1()` | Select and read the corresponding ADC channels. |
| `lcd_init()` | Initializes the LCD in 4-bit mode. |
| `lcd_nibble()` | Sends a four-bit value to the LCD. |
| `lcd_cmd()` | Sends an LCD command. |
| `lcd_data()` | Sends a character to the LCD. |
| `lcd_string()` | Displays a string. |
| `lcd_num()` | Displays a numerical ADC value as digits. |
| `MSDelay()` | Provides a Timer 0-based delay; actual timing depends on the oscillator and timer configuration. |
| `delay()` | Provides a software delay loop. |
| `led_buzzer()` | Controls LEDs and the buzzer according to the programmed conditions. |

## Challenges and Technical Considerations


- **Conversion timing:** The firmware must use the EOC signal and ADC control signals in the correct sequence before reading the result.
- **Sequential readings:** The ADC0808 reads one selected channel at a time, so the sensor values are collected one after the other.
- **Output-control logic:** Threshold conditions must be ordered correctly and should define the intended behavior when the two sensor readings differ.
- **Timing:** Timer delays depend on the crystal frequency and timer reload values.


## Simulation Results

## Physical Hardware

//IMAGE

## Sensor Baseline and Threshold Selection

During the original development, clean-air sensor readings were observed and used to choose experimental thresholds for the LED and buzzer logic. The original numerical readings and threshold-selection records are no longer available, so specific historical values are not reported here.

These thresholds are prototype settings, not official AQI breakpoints. Clean-air observations alone do not establish accurate gas concentrations. Further characterization and comparison with a suitable reference instrument would be needed before making calibrated measurement claims.

## Build and Simulate

### Build in Keil µVision

1. Open or create the Keil project.
2. Select the microcontroller that matches the actual circuit.
3. Add the firmware source file(s).
4. Confirm the oscillator and project settings.
5. Compile the project and resolve build errors.
6. Generate a HEX file if required by the Proteus model.

### Simulate in Proteus

1. Open the Proteus project.
2. Load the compiled HEX file into the microcontroller model.
3. Confirm the microcontroller clock setting and the ADC0808 clock/reference connections against the schematic.
4. Check the sensor inputs or use adjustable analog sources if that is how the simulation is configured.
5. Run the simulation and observe ADC readings, LCD output, and LED/buzzer behavior.


## Limitations

- The LCD displays raw ADC counts rather than calibrated concentrations or official AQI values.
- The original clean-air readings and threshold-selection records are unavailable.
- The physical hardware has not been re-verified recently.
- The exact microcontroller and sensor models, as well as ADC clock and reference connections, must be confirmed from the final circuit.
- The thresholds have not been validated against a reference instrument.
- The green LED pin assignment must be resolved if it conflicts with ADC0808 OE on P3.6.

## Future Improvements

- Re-test the physical circuit and document the results.
- Confirm the exact sensor and microcontroller models and finalize the pin map.
- Collect and document new sensor readings under repeatable conditions.
- Validate sensor behavior against a suitable reference instrument.
- Evaluate averaging or filtering to reduce fluctuations, if needed.
- Review and test the alert logic with different combinations of sensor readings.


