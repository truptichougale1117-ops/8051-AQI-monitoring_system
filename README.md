# Air Quality Monitoring System using 8051 and ADC0808

A student embedded-systems project. An AT89C51-compatible 8051 microcontroller reads two analog sensor inputs through an ADC0808, shows the readings on a 16×2 LCD, and turns on LEDs and a buzzer based on threshold values in the code.

> **Important:** The numbers shown on the LCD are raw 8-bit ADC counts (0–255). They are not calibrated air-quality measurements and they are not AQI values.

## Project Status

**Implemented (in the code and circuit design)**
- Reading two ADC0808 channels one after the other
- Showing both readings on a 16×2 LCD in 4-bit mode
- Controlling three LEDs (green, yellow, red) and a buzzer from the readings
- Timer 0 delays using an 11.0592 MHz crystal

**Tested**
- [Fill in only what you have actually tested, and whether it was in Proteus or on hardware. See the Testing section.]

**Not done yet**
- Sensor calibration
- Checking the thresholds against a reference instrument
- Filtering or averaging of readings

## Hardware

- AT89C51-compatible 8051 microcontroller, 11.0592 MHz crystal
- ADC0808 (8-bit, 8-channel, with a multiplexed analog input)
- Two analog sensors: [sensor models]
- 16×2 LCD (4-bit mode)
- Red, yellow, and green LEDs
- Buzzer
- Resistors, wiring, and power supply: [values and ratings]

## Pin Connections

**ADC0808 to 8051**

| ADC0808 | 8051 |
|---|---|
| D0–D7 | P1.0–P1.7 |
| Address A, B, C | P3.0, P3.1, P3.2 |
| ALE | P3.3 |
| START | P3.4 |
| EOC | P3.5 |
| OE | P3.6 |

ADC0808 clock source: [fill in]. VREF connection: [fill in]. Supply connection: [fill in].

**16×2 LCD to 8051**

| LCD | 8051 |
|---|---|
| RS | P2.2 |
| Enable | P2.3 |
| D4–D7 | P2.4–P2.7 |
| RW | Ground (write only) |

Only D4–D7 are connected because the LCD runs in 4-bit mode.

**LEDs and buzzer**

| Output | 8051 |
|---|---|
| Red LED | P2.0 |
| Yellow LED | P2.1 |
| Green LED | [pin] |
| Buzzer | P3.7 |

The ADC0808 OE signal already uses P3.6, so the green LED could not use that pin and is connected to [pin] instead.

## How It Works

1. The two sensors give analog voltages to two input channels of the ADC0808.
2. For each reading, the 8051 sets the address lines A, B, C to pick the channel and pulses ALE to latch the address.
3. The 8051 pulses START to begin the conversion, then waits for EOC to show that the conversion is finished.
4. The 8051 raises OE so the ADC puts the 8-bit result on D0–D7, and reads it from Port 1.
5. The program reads channel 1 (`ADC_Channel1()`), then channel 0 (`ADC_Channel0()`).
6. Both values are shown on the LCD.
7. `led_buzzer()` compares the values with the thresholds and switches the LEDs and buzzer.

```text
Analog sensors -> ADC0808 -> 8051 -> LCD + LEDs + buzzer
```

### LCD display

The LCD is used in 4-bit mode, so each byte is sent as two nibbles, high nibble first. `lcd_nibble()` does this. `lcd_cmd()` and `lcd_data()` use it to send commands and characters. `lcd_num()` splits a value from 0 to 255 into hundreds, tens, and ones so each digit can be shown as a character.

The display looks like this (the numbers here are only an example of the format):

```text
S1:125
S2:087
```

[Fill in: which ADC channel is S1 and which is S2, and which sensor is connected to each.]

### LED and buzzer logic

The thresholds in the code are about 100, 150, and 200 ADC counts. I chose them for demonstration with this prototype. They are not official AQI breakpoints, and I have not checked them against a reference instrument.

| Condition | Green | Yellow | Red | Buzzer |
|---|---|---|---|---|
| [fill in from `led_buzzer()`] | | | | |
| [fill in from `led_buzzer()`] | | | | |
| [fill in from `led_buzzer()`] | | | | |

## Code Structure

All the firmware is in `Code/main.c`.

| Function | What it does |
|---|---|
| `main()` | Sets up the LCD and ADC control pins, reads both channels, updates the LCD, calls `led_buzzer()` |
| `ADC_Channel1()`, `ADC_Channel0()` | Select a channel, start a conversion, wait for EOC, return the result |
| `lcd_init()` | Initializes the LCD in 4-bit mode |
| `lcd_nibble()`, `lcd_cmd()`, `lcd_data()`, `lcd_string()` | Send nibbles, commands, characters, and strings to the LCD |
| `lcd_num()` | Shows a 0–255 value as three digits |
| `MSDelay()` | Millisecond delay using Timer 0 (depends on the 11.0592 MHz crystal) |
| `delay()` | Simple software delay loop, less accurate than `MSDelay()` |
| `led_buzzer()` | Sets the LEDs and buzzer from both sensor readings |

## Build and Simulate

**Build in Keil µVision**
1. Create a project and choose the AT89C51 (or a compatible device).
2. Add `Code/main.c`.
3. Set the crystal frequency to 11.0592 MHz.
4. Turn on "Create HEX File" and build.

**Simulate in Proteus**
1. Open the project from the `Proteus/` folder.
2. Load the HEX file into the microcontroller and set its clock to 11.0592 MHz.
3. Give the ADC0808 a clock signal and set VREF: [clock source and VREF value].
4. Connect the sensors, or use adjustable analog sources instead: [what you used].
5. Run the simulation and watch the LCD, LEDs, and buzzer.

## Testing

Only mark a test as done if you have actually checked it.

| Test | Where tested (Proteus / hardware) | Result |
|---|---|---|
| ADC channel selection and conversion | [ ] | [result or "not tested"] |
| Reading both channels | [ ] | [result or "not tested"] |
| LCD output | [ ] | [result or "not tested"] |
| Each LED works | [ ] | [result or "not tested"] |
| Buzzer works | [ ] | [result or "not tested"] |
| LED and buzzer response to different inputs | [ ] | [result or "not tested"] |

Images (add only the ones that exist in `Images/`):
- Circuit diagram: `Images/circuit.png`
- LCD output: `Images/output.png`
- Block diagram: `Images/block_diagram.png`

## Challenges and Lessons Learned

- **Pin conflict:** P3.6 was already used for the ADC0808 OE signal, so I had to move the green LED to another pin. This made me plan the pin use of every port before wiring.
- [Add other real problems you faced, such as LCD not initializing, wrong readings, or timing issues, and how you fixed them. Delete this line if there are none.]

## Limitations

- The two channels are read one after the other, not at the same time.
- The readings are raw ADC counts, not calibrated concentrations or AQI.
- Accuracy depends on the sensors and the environment.
- The thresholds are development values and have not been validated.

## Possible Next Steps

These are not implemented yet.
- Calibrate the sensors using reference measurements
- Average several readings to reduce noise
- Replace the simple thresholds with a properly validated classification

## Repository Structure

```text
Code/main.c          Firmware
Proteus/             Simulation files
Images/              Circuit diagram, LCD output, block diagram
README.md
```

## License

Not chosen yet. [Add the license name here after you pick one and add the LICENSE file.]
