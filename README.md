# CAN Gimbal Control System

A 2-axis gimbal control system built on STM32 + CAN bus, with a Linux-based debug/OTA tool chain.

> **Status:** Active development — Phase 0 (hardware bring-up) in progress.

## Project Overview

This project implements a CAN-networked gimbal controller: two STM32 nodes (one per axis) running closed-loop motor control, connected through CAN bus to a Linux debug/validation tool on a Raspberry Pi. The Linux side provides telemetry logging, automated testing, and over-the-air firmware updates via a custom CAN OTA protocol.

This is a personal learning and portfolio project, not a production system.

## Motivation

- Hands-on experience with STM32 peripheral programming and CAN bus
- Design and implement a custom bootloader with CAN-based OTA firmware update
- Practice RTOS-based embedded architecture on real hardware
- Build a Linux-side debug/validation tool using SocketCAN
- Create a tangible firmware engineering portfolio piece

## System Architecture

```
                     Linux Debug Tool (Raspberry Pi)
                            |
       +--------------------+--------------------+
       |                    |                    |
  Test Runner        Telemetry Logger      Firmware Manager
       |                    |                    |
       +--------------------+--------------------+
                            |
                        SocketCAN (slcan0)
                            |
                      ESP32-S3 USB-CAN Bridge
                       (SLCAN firmware)
                            |
                     CAN Bus (250 kbps)
                    +-------+-------+
                    |               |
                 Axis X          Axis Y
                 STM32           STM32
              (Node 0x01)     (Node 0x02)
                    |               |
             +------+------+ +------+------+
             | Bootloader  | | Bootloader  |
             +-------------+ +-------------+
             | Application | | Application |
             +------+------+ +------+------+
                    |               |
             Control Loop      Control Loop
                    |               |
             Mag Encoder       Mag Encoder
                    |               |
                  Motor            Motor
```

## Hardware

| Component | Qty | Purpose |
|-----------|-----|---------|
| STM32 NUCLEO-F446RE | 2 | Axis X / Axis Y motor control nodes |
| ESP32-S3 | 1 | USB-CAN bridge (TWAI + SLCAN firmware) |
| SN65HVD230 CAN transceiver | 3 | One per CAN node |
| Raspberry Pi | 1 | Linux upper computer (debug + OTA) |
| 120 Ohm termination resistor | 2 | CAN bus termination (each end) |
| NEMA17 + TMC2209 | 2 | Stepper motors (one per axis) |
| Magnetic encoder | 2 | Position feedback (one per axis) |
| BMP280 | 1 | Temperature + barometric pressure sensor |
| TOF400C (VL53L1X) | 1 | Time-of-flight distance sensor |

## CAN Network

| Node | Node ID | Role |
|------|---------|------|
| STM32 Axis X | 0x01 | Motor control + sensor acquisition |
| STM32 Axis Y | 0x02 | Motor control + sensor acquisition |
| Linux Debug (via ESP32-S3 bridge) | 0x03 | Telemetry, testing, OTA firmware update |

**Bitrate:** 250 kbps

**Full CAN protocol:** See [`docs/can/CAN_PROTOCOL.md`](docs/can/CAN_PROTOCOL.md)

## CAN OTA Firmware Update

The system includes a custom bootloader on each STM32 node and a 3-phase OTA protocol over CAN:

1. **Handshake** — Linux sends firmware size, CRC32, version; STM32 erases APP flash region and ACKs
2. **Data transfer** — Firmware binary chunked into 6-byte CAN frames with sequence numbers, ACK per frame
3. **Verification** — CRC32 check on received image; reboot to new application on success

| CAN ID | Message | Direction |
|--------|---------|-----------|
| 0x100 | UPDATE_START | Linux -> STM32 |
| 0x101 | UPDATE_DATA | Linux -> STM32 |
| 0x102 | UPDATE_END | Linux -> STM32 |
| 0x103 | UPDATE_ACK | STM32 -> Linux |
| 0x104 | UPDATE_NACK | STM32 -> Linux |
| 0x105 | VERSION_REQUEST | Linux -> STM32 |
| 0x106 | VERSION_RESPONSE | STM32 -> Linux |
| 0x107 | REBOOT | Linux -> STM32 |

**Full OTA protocol spec:** See [`docs/architecture/SYSTEM_ARCHITECTURE.md`](docs/architecture/SYSTEM_ARCHITECTURE.md)

## Flash Layout (per STM32 node)

```
0x08000000  +---------------------------+
            |       Bootloader          |
            |      (32KB, 0x8000)       |
0x08008000  +---------------------------+
            |       Application         |
            |    (remaining flash)      |
            +---------------------------+
```

The application linker script sets `FLASH ORIGIN = 0x08008000`, and `main()` relocates the vector table with `SCB->VTOR = 0x08008000` as its first operation.

## Software Stack

- **MCU:** STM32F446RE (ARM Cortex-M4)
- **RTOS:** FreeRTOS
- **Communication:** CAN 2.0B, 250 kbps
- **Build system:** PlatformIO + STM32Cube HAL
- **Linux tools:** Python 3, SocketCAN
- **Language:** C (firmware), Python (Linux tools)

## Repository Structure

```
CAN-Gimbal-Control/
├── docs/
│   ├── architecture/      # System architecture & OTA protocol
│   ├── can/               # CAN message definitions
│   ├── hardware/          # Wiring, pin maps, schematics
│   └── testing/           # Test reports
├── ecu_1/                 # Axis X STM32 node
│   ├── Core/              # main, system init, interrupts
│   ├── Drivers/           # Peripheral drivers (GPIO, Timer, I2C)
│   ├── Application/       # Control loop logic
│   ├── RTOS/              # FreeRTOS tasks and config
│   └── CAN/               # CAN TX/RX, message handling
├── ecu_2/                 # Axis Y STM32 node (same structure)
├── ecu_3/                 # Reserved
├── common/                # Shared CAN IDs, types, utilities
├── platformIO/            # PlatformIO build configuration
├── stm32_Util/            # Reusable hardware utility drivers
├── tests/                 # Test scripts
└── tools/                 # Linux debug tool, OTA scripts
```

## Development Roadmap

See [`DEVELOPMENT_PLAN.md`](DEVELOPMENT_PLAN.md) for the full phased plan.

| Phase | Topic | Status |
|-------|-------|--------|
| 0 | Hardware bring-up (LED blink, PlatformIO) | **In Progress** |
| 1 | STM32 GPIO + basic peripherals | Planned |
| 2 | Timer / Interrupt | Planned |
| 3 | Sensor drivers (BMP280, VL53L1X) | Planned |
| 4 | CAN basic communication | Planned |
| 5 | Multi-node CAN | Planned |
| 6 | CAN message protocol | Planned |
| 7 | FreeRTOS integration | Planned |
| 8 | Bootloader + CAN OTA | Planned |
| 9 | Motor control loop | Planned |
| 10 | Linux debug tool (SocketCAN) | Planned |
| 11 | Testing / documentation / portfolio | Planned |

## Future Work (V2)

- Batch ACK for faster OTA transfer
- ESP8266 wireless OTA path
- Real-time telemetry dashboard (ESP32)
- Automated test runner with fault injection
- Telemetry data logger

## Out of Scope

- UDS / ISO-TP / DoIP
- Secure Boot / cryptographic signing
- A/B firmware partitions
- Edge AI
- Cloud backend

## Author

Hongyi Mei
