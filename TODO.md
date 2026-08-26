# Project TODO

Master checklist of decisions and tasks. Check off as you go.

## Decisions Needed Before Starting

- [ ] **Project name** — choose a name and update all [PROJECT_NAME] placeholders
- [ ] **STM32 model** — decide which STM32 board(s) to use, update [STM32_MODEL]
- [ ] **CAN transceivers** — select CAN transceiver ICs, update [CAN_TRANSCEIVER_MODEL_x]
- [ ] **Sensors** — confirm TOF and temperature sensor models
- [ ] **Motor** — decide on motor type and model
- [ ] **LED** — decide on LED type (single, RGB, strip, etc.)
- [ ] **Build system** — STM32CubeIDE project? Makefile? CMake?
- [ ] **CAN bitrate** — 125k / 250k / 500k / 1M?
- [ ] **ECU node names and purposes** — define what each ECU does
- [ ] **ECU node inputs/outputs** — define peripherals per node

## Phase Checklist

- [ ] Phase 0 — Hardware bring-up
- [ ] Phase 1 — STM32 GPIO
- [ ] Phase 2 — Timer / Interrupt
- [ ] Phase 3 — Sensor driver
- [ ] Phase 4 — CAN basic communication
- [ ] Phase 5 — Multi-node CAN
- [ ] Phase 6 — CAN message protocol
- [ ] Phase 7 — FreeRTOS
- [ ] Phase 8 — ECU application architecture
- [ ] Phase 9 — Fault detection
- [ ] Phase 10 — Testing / debugging
- [ ] Phase 11 — Documentation / portfolio

## Things to Think About

- [ ] How will you power 3 MCU boards + sensors + motor simultaneously?
- [ ] Do you need level shifters for any sensor interfaces?
- [ ] How will you terminate the CAN bus? (120 ohm resistors)
- [ ] What is your debugging strategy? (SWD? UART printf? SWO?)
- [ ] How will you version control STM32CubeIDE generated files?
- [ ] Do you want to use HAL, LL, or bare register access? (Consider doing register-level for learning)
