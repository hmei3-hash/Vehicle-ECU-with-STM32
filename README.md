# Vehicle-ECU-with-STM32

A multi-node CAN-based ECU learning system built on STM32 + FreeRTOS.

> **Status:** Work in progress — this is a personal learning and portfolio project.

## Project Overview

This project implements a small-scale, multi-node automotive ECU system for learning embedded software development. The system consists of three ECU nodes communicating over a CAN bus, each responsible for different vehicle subsystem functions.

This is **not** a safety-critical or production-grade ECU. It is a hands-on learning project designed to build real skills in automotive embedded systems.

## Motivation

- Develop practical experience with STM32 MCU peripheral programming
- Learn CAN bus communication and automotive-style distributed systems
- Practice RTOS-based embedded software architecture
- Build a tangible portfolio piece demonstrating firmware engineering skills

## System Architecture

```
  [ECU Node 1]          [ECU Node 2]          [ECU Node 3]
  Sensor Node          Control Node          Dashboard Node
       |                     |                     |
       +---------------------+---------------------+
                         CAN Bus
                     [CAN_BITRATE]
```

**Architecture details:** See `docs/architecture/` for system diagrams and design documents.

## Hardware

| Component            | Model / Part Number         |
|----------------------|-----------------------------|
| MCU                  | STM32 NUCLEO-F446RE               |
| CAN Transceiver 1   | WCMCU-20   |
| CAN Transceiver 2   | WCMCU-20   |
| CAN Transceiver 3   | WCMCU-20   |
| TOF Sensor           | TOF400C (VL53L1X)          |
| Temperature Sensor   | BMP280 (temperature + barometric pressure) (temperature + barometric pressure)  |
| LED                  | Single LED             |
| Motor                | NEMA17 with TMC2209               |
| Logic Analyzer       | [LOGIC_ANALYZER_MODEL]      |
| USB Connection       | [USB_CONNECTION_TYPE]        |
| Other                | [OTHER_HARDWARE]            |

**Pin mappings:** [TBD] — See `docs/hardware/` once defined.

## Software Stack

- **MCU:** STM32 NUCLEO-F446RE
- **RTOS:** FreeRTOS
- **Communication:** CAN 2.0 [TBD]
- **Build system:** [TBD] — STM32CubeIDE / Makefile / CMake
- **Debugger:** [TBD]
- **Language:** C

## ECU Nodes

### ECU Node 1 — Sensor Node

- **Purpose:** Read TOF400C (VL53L1X) and BMP280 (temperature + barometric pressure) sensors, transmit data over CAN
- **Inputs:** TOF400C (I2C), BMP280 (temperature + barometric pressure) (I2C)
- **Outputs:** CAN TX (sensor data)
- **Peripherals:** I2C, CAN, Timer
- **CAN responsibilities:** Transmit sensor readings periodically

### ECU Node 2 — Control Node

- **Purpose:** Receive sensor data via CAN, execute control logic, drive NEMA17 motor
- **Inputs:** CAN RX (sensor data)
- **Outputs:** NEMA17 with TMC2209 (STEP/DIR or UART), CAN TX (status)
- **Peripherals:** CAN, GPIO, Timer, UART [TBD]
- **CAN responsibilities:** Receive sensor data, transmit control status

### ECU Node 3 — Dashboard Node

- **Purpose:** Receive data from all nodes via CAN, display system status (UART now, LCD later)
- **Inputs:** CAN RX (sensor data, control status)
- **Outputs:** UART serial print (future: LCD), Single LED status indicator
- **Peripherals:** CAN, UART, GPIO
- **CAN responsibilities:** Receive all messages, no TX [TBD]

## CAN Network

- **Bitrate:** [CAN_BITRATE]
- **Nodes:** [CAN_NODE_LIST]

### Message Definitions

| Message     | CAN ID   | Sender | Receiver | Period   | DLC   | Payload   |
|-------------|----------|--------|----------|----------|-------|-----------|
| [MESSAGE_1] | [CAN_ID] | [NODE] | [NODE]   | [PERIOD] | [DLC] | [PAYLOAD] |
| [MESSAGE_2] | [CAN_ID] | [NODE] | [NODE]   | [PERIOD] | [DLC] | [PAYLOAD] |
| [MESSAGE_3] | [CAN_ID] | [NODE] | [NODE]   | [PERIOD] | [DLC] | [PAYLOAD] |

**Full CAN protocol specification:** See `docs/can/CAN_PROTOCOL.md` once defined.

## Development Roadmap

See [DEVELOPMENT_PLAN.md](DEVELOPMENT_PLAN.md) for the full phased plan.

| Phase | Topic                        | Status  |
|-------|------------------------------|---------|
| 0     | Hardware bring-up            | [TBD]   |
| 1     | STM32 GPIO                   | [TBD]   |
| 2     | Timer / Interrupt            | [TBD]   |
| 3     | Sensor driver                | [TBD]   |
| 4     | CAN basic communication      | [TBD]   |
| 5     | Multi-node CAN               | [TBD]   |
| 6     | CAN message protocol         | [TBD]   |
| 7     | FreeRTOS                     | [TBD]   |
| 8     | ECU application architecture | [TBD]   |
| 9     | Fault detection              | [TBD]   |
| 10    | Testing / debugging          | [TBD]   |
| 11    | Documentation / portfolio    | [TBD]   |

## Testing

See [TEST_PLAN.md](TEST_PLAN.md) for test case templates and results.

## Repository Structure

```
Vehicle-ECU-with-STM32/
├── docs/
│   ├── architecture/      # System design documents
│   ├── can/               # CAN protocol specs
│   ├── hardware/          # Schematics, pin maps, wiring
│   └── testing/           # Test reports and logs
├── ecu_1/                 # ECU Node 1
│   ├── Core/              # main, system init, interrupts
│   ├── Drivers/           # Peripheral drivers (GPIO, ADC, PWM, etc.)
│   ├── Application/       # ECU application logic
│   ├── RTOS/              # FreeRTOS tasks and config
│   └── CAN/               # CAN TX/RX, message handling
├── ecu_2/                 # ECU Node 2 (same structure)
├── ecu_3/                 # ECU Node 3 (same structure)
├── common/                # Shared code (CAN IDs, types, utilities)
├── tests/                 # Test scripts and harnesses
└── tools/                 # Helper scripts, analysis tools
```

## Future Work

- [TBD] — Diagnostics (UDS / OBD-II learning)
- [TBD] — Power management / sleep modes
- [TBD] — Bootloader / firmware update over CAN
- [TBD] — Hardware-in-the-loop testing
- [TBD] — PCB design for custom ECU board

## License

[TBD]

## Author

Hongyi Mei
