# Water Level Monitoring System – Arduino Nano + TLE4946

## Overview

This project implements a water level monitoring system based on three TLE4946 Hall sensors connected to an Arduino Nano.

The system:

- Reads three Hall sensors in analog mode
- Converts ADC values to real voltage
- Applies voltage-based hysteresis
- Controls three relays
- Logs sensor voltages via UART
- Measures real Vcc at startup
- Uses watchdog for software protection

---

## Hardware Configuration

### Microcontroller

- Arduino Nano (ATmega328P)
- Powered via USB
- Measured Vcc is read at startup using internal 1.1V reference

### Hall Sensors

- Type: TLE4946
- Output: Open collector
- Supply voltage: 3.37 V (from Nano 3V3 pin)
- Pull-up resistor: 3k to 3.37 V

### Signal Conditioning

Each Hall channel includes:

- 1 nF capacitor to GND (local EMC filtering)
- 1k series resistor
- 100 nF capacitor to GND (low-pass filtering)
- Additional 1k resistor at ADC input
- BAV199 clamp diodes to Vcc and GND

This provides:

- EMI suppression
- ADC input protection
- Overvoltage protection
- Low source impedance for ADC stability

---

## Pin Assignment

| Function            | Arduino Pin |
|---------------------|------------|
| LOW level sensor    | A3         |
| HIGH level sensor   | A6         |
| SAFE level sensor   | A7         |
| LOW relay           | A0         |
| HIGH relay          | A1         |
| SAFE relay          | A2         |

---

## Software Architecture

### 1. Vcc Measurement

At startup, the system measures actual Vcc using the internal 1.1V reference.

This ensures:

- Correct voltage conversion
- Independence from USB voltage fluctuations
- Accurate ADC-to-voltage scaling

---

### 2. ADC Reading

Each analog channel:

- Discards the first ADC reading (MUX stabilization)
- Uses the second reading for measurement

---

### 3. Voltage Conversion

ADC values are converted to voltage using:

V = (ADC / 1023) × Vcc

---

### 4. Hysteresis

Voltage-based hysteresis is applied:

- LOW threshold: 0.3 V
- HIGH threshold: 1.5 V

State change logic:

- LOW → HIGH if voltage > 1.5 V
- HIGH → LOW if voltage < 0.3 V

This prevents relay chatter and ensures stable operation.

---

### 5. Watchdog

- Enabled with 1 second timeout
- Reset in every loop cycle
- Protects against software lockups

---

## System Behavior

At startup:

1. Watchdog is disabled
2. Vcc is measured
3. System waits for stabilization
4. Initial Hall states are read
5. Relays are set accordingly
6. Watchdog is enabled

During operation:

- Sensors are sampled continuously
- Voltage values are logged every 500 ms
- Relays follow hysteresis-controlled states

---

## Expected Voltage Levels

Typical values:

- LOW state: ~0 V
- HIGH state: ~3.37 V
- Arduino Vcc: ~4.2 V (USB powered)

Maximum ADC value for HIGH state:

(3.37 / 4.2) × 1023 ≈ 816

---

## Safety Notes

- Hall sensors are always exposed to magnetic field at startup
- System does not rely on undefined magnetic state
- Hardware filtering ensures EMI resistance
- Clamp diodes protect ADC inputs

---

## Future Improvements (Optional)

- Noise RMS monitoring
- Sensor diagnostic (open wire detection)
- Fail-safe logic for SAFE channel
- EEPROM reset logging

---

## Status

Current version provides stable water level detection and relay control.

Ready for long-duration testing.
