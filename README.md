# AC400 IR Controller – Sender Configuration

This repository contains the `sender.yaml` configuration file used to transmit infrared (IR) commands for controlling an **Record Power AC400 air cleaner**.
The file defines IR command sequences that replicate the functions of the original remote control and can be used with microcontrollers and IR transmitter hardware.

## File Overview
The YAML file `sender.yaml` contains:
- Infrared protocol definitions
- Raw or encoded IR command data
- Command mappings for air cleaner functions such as:
  - Power off
  - Fan speed levels
  - Timer modes

Just copy the content of sender.yaml in your existing ESPHome device configurartion.

## Requirements
Typical hardware and software requirements:
- ESP8266 or ESP32 microcontroller
- IR LED (with appropriate resistor)
- Firmware or framework that supports IR transmission (e.g. ESPHome)
- Configured GPIO pin for the IR transmitter

# Custom Infrared Protocol Documentation
## Overview
This document describes a **custom infrared (IR) remote control protocol** that was reverse-engineered using PulseView and logic analysis.  
The protocol is **not NEC-compatible**, but shares several structural concepts such as inverted fields and frame repetition.

## 1. Physical Layer
### 1.1 Signal Source
- Signal observed at the output of a demodulating IR receiver (e.g. TSOP-style)
- Therefore:
  - LOW = IR carrier present
  - HIGH = no IR carrier

### 1.2 Carrier
- IR carrier frequency: 38 kHz
- Carrier duty cycle: ~50%

### 1.3 Timing Units (Measured)
| Name        | Duration (µs) | Meaning                          |
|-------------|---------------|----------------------------------|
| `PULSE`     | ~402          | Fixed IR burst                   |
| `SEPARATOR` | ~759          | Short pause before a logical `1` |
| `ZERO`      | ~1258         | Pause encoding logical `0`       |
| `ONE`       | ~1597         | Pause encoding logical `1`       |
| `LEADER`    | ~8840         | Initial synchronization gap      |
| `PAUSE`     | ~4296         | Pause after leader               |

All values tolerate moderate jitter (±10–15%).

## 2. Encoding
Bits are not encoded as “pulse + pause”. Instead, bits are defined by rules about the sequence of pauses and pulses. This leads to two invariant observations:

- After every logical `0`, a `PULSE` always occurs
- Before every logical `1`, a `SEPARATOR` always occurs
- Pulses act as structural markers, not data
- Information is carried primarily by pause length and ordering
- The protocol is self-synchronizing and tolerant to jitter

| Logical Bit | Encoding Sequence            |
|------------|-------------------------------|
| `0`        | `ZERO` → `PULSE`              |
| `1`        | `SEPARATOR` → `ONE`           |

## 3. Frame Structure
### 3.1 High-Level Frame Layout
A complete transmission consists of:

- LEADER
- PAUSE
- FRAME
- REPEAT_SYNC
- FRAME

The leader appears only once, followed by two identical frames.

### 3.2 Frame Layout (Logical)
Each frame is composed of:

- ADDR (16 bits)
- CMD (8 bits)
- ~CMD (8 bits, bitwise inverted)
- REST (24 bits, fixed)

Total per frame: **56 bits**

### 3.3 Address Field
`ADDR = 00000000 11111111`

- Fixed value
- Second byte is the bitwise inverse of the first
- Serves as a device or protocol identifier

### 3.4 Command Field
- CMD = 8-bit command
- ~CMD = bitwise inversion of CMD
- Simple but effective **error detection**
- If any bit is corrupted, `CMD != ~CMD`
- Eliminates the need for a CRC

Example:
```
CMD = 00001001
~CMD = 11110110
```

### 3.5 REST Field (Footer)
`REST = 001000000000110100000100`

- Constant for all commands
- Not a CRC, not a counter, not inverted, does not depend on CMD
- Most likely meaning: Device/model identifier, protocol version marker or frame tail synchronization pattern

### 3.6 Repeat Synchronization
`REPEAT_SYNC = 11111111`

- Separates the first and second frame
- No new leader is sent
- Improves reliability and key-hold behavior

## 4. Complete Logical Frame
```
ADDR
CMD
~CMD
REST
REPEAT_SYNC
ADDR
CMD
~CMD
REST
```

## 5. Repeat Behavior
- Each button press transmits two identical frames
- Only one leader is used
- Holding a button repeats the same pattern
- No toggle bit or counter is present
