# 8051-AQI-monitoring_system

This project is a microcontroller-based air quality monitoring system
using the 8051 microcontroller and ADC0808.

The system is designed to acquire analog signals from two gas/air-quality
sensors. Since the 8051 does not have a built-in ADC, the ADC0808 is used
to convert the analog sensor signals into 8-bit digital data.

The 8051 selects the required ADC channel, starts the conversion,
waits for the conversion to complete, and reads the digital output
through Port 1. 
