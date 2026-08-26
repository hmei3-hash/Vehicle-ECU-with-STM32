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
  [ECU_1_NAME]          [ECU_2_NAME]          [ECU_3_NAME]
       |                     |                     |
       +---------------------+---------------------+
                         CAN Bus
                     [CAN_BITRATE]
```

**Architecture details:** See `docs/architecture/` for system diagrams and design documents.

## Hardware

| Component            | Model / Part Number         |
|----------------------|-----------------------------|
| MCU                  | [STM32_MODEL]               |
| CAN Transceiver 1   | [CAN_TRANSCEIVER_MODEL_1]   |
| CAN Transceiver 2   | [CAN_TRANSCEIVER_MODEL_2]   |
| CAN Transceiver 3   | [CAN_TRANSCEIVER_MODEL_3]   |
| TOF Sensor           | [TOF_SENSOR_MODEL]          |
| Temperature Sensor   | [TEMPERATURE_SENSOR_MODEL]  |
| LED                  | [LED_COMPONENT]             |
| Motor                | [MOTOR_MODEL]               |
| Logic Analyzer       | [LOGIC_ANALYZER_MODEL]      |
| USB Connection       | [USB_CONNECTION_TYPE]        |
| Other                | [OTHER_HARDWARE]            |

**Pin mappings:** [TBD] — See `docs/hardware/` once defined.

## Software Stack

- **MCU:** [STM32_MODEL]
- **RTOS:** FreeRTOS
- **Communication:** CAN 2.0 [TBD]
- **Build system:** [TBD] — STM32CubeIDE / Makefile / CMake
- **Debugger:** [TBD]
- **Language:** C

## ECU Nodes

### ECU Node 1 — [ECU_1_NAME]

- **Purpose:** [ECU_1_PURPOSE]
- **Inputs:** [ECU_1_INPUTS]
- **Outputs:** [ECU_1_OUTPUTS]
- **Peripherals:** [ECU_1_PERIPHERALS]
- **CAN responsibilities:** [ECU_1_CAN_RESPONSIBILITIES]

### ECU Node 2 — [ECU_2_NAME]

- **Purpose:** [ECU_2_PURPOSE]
- **Inputs:** [ECU_2_INPUTS]
- **Outputs:** [ECU_2_OUTPUTS]
- **Peripherals:** [ECU_2_PERIPHERALS]
- **CAN responsibilities:** [ECU_2_CAN_RESPONSIBILITIES]

### ECU Node 3 — [ECU_3_NAME]

- **Purpose:** [ECU_3_PURPOSE]
- **Inputs:** [ECU_3_INPUTS]
- **Outputs:** [ECU_3_OUTPUTS]
- **Peripherals:** [ECU_3_PERIPHERALS]
- **CAN responsibilities:** [ECU_3_CAN_RESPONSIBILITIES]

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
