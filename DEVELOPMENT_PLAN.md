# Development Plan

This document defines the phased learning roadmap for the ECU project. Each phase has clear objectives, tasks, acceptance criteria, and test methods.

**Rule:** Do not skip phases. Each phase builds on the previous one. Complete the acceptance criteria before moving on.

---

## Phase 0 — Hardware Bring-up

**Learning objectives:**
- Verify all hardware components are functional
- Set up the development environment (toolchain, debugger, IDE)
- Confirm basic MCU connectivity (flash, debug, serial output)

**What you need to do:**
- [ ] Unbox and visually inspect all components
- [ ] Set up STM32 development environment: [TBD — STM32CubeIDE / VS Code + toolchain]
- [ ] Connect STM32 NUCLEO-F446RE to PC via [USB_CONNECTION_TYPE]
- [ ] Flash a minimal blink LED program (bare-metal, no HAL if possible)
- [ ] Verify serial output (UART printf or SWO)
- [ ] Document your hardware setup with photos
- [ ] Record pin connections in `docs/hardware/`

**Acceptance criteria:**
- [ ] LED blinks on the target board
- [ ] printf output visible over serial/SWO
- [ ] Debugger can halt, step, and inspect variables
- [ ] Development environment documented

**Recommended test methods:**
- Visual confirmation of LED blink
- Serial terminal output check
- Debugger breakpoint test

---

## Phase 1 — STM32 GPIO

**Learning objectives:**
- Understand GPIO registers (MODER, ODR, IDR, PUPDR, etc.)
- Configure GPIO as input and output
- Understand push-pull vs open-drain, pull-up/pull-down

**What you need to do:**
- [ ] Configure GPIO pins as output — drive Single LED
- [ ] Configure GPIO pins as input — read a button or jumper wire
- [ ] Implement GPIO toggling at different speeds
- [ ] Read input state and reflect on output (input-to-LED)
- [ ] Document which registers you configured and why

**Acceptance criteria:**
- [ ] Can control GPIO output (LED on/off) by writing to registers
- [ ] Can read GPIO input and react to it
- [ ] Code is register-level (not just HAL calls) — you understand what happens underneath
- [ ] GPIO configuration documented in code comments

**Recommended test methods:**
- Multimeter on GPIO output pins
- Logic analyzer capture of GPIO toggling
- Manual button press → LED response

---

## Phase 2 — Timer / Interrupt

**Learning objectives:**
- Configure STM32 timer peripherals
- Understand prescaler, auto-reload, update events
- Set up timer interrupts (NVIC)
- Understand interrupt priorities and nesting

**What you need to do:**
- [ ] Configure a timer to generate periodic interrupts
- [ ] Toggle an LED in the timer ISR at a precise frequency
- [ ] Measure the actual frequency with [LOGIC_ANALYZER_MODEL]
- [ ] Configure an external interrupt (EXTI) on a GPIO pin
- [ ] Experiment with interrupt priority levels
- [ ] Document timer calculations (clock source, prescaler, period)

**Acceptance criteria:**
- [ ] Timer interrupt fires at the expected frequency (verified by measurement)
- [ ] External interrupt triggers correctly on edge/level
- [ ] Can explain the relationship between clock, prescaler, ARR, and output frequency
- [ ] ISR is minimal — no blocking code inside

**Recommended test methods:**
- Logic analyzer frequency measurement
- Oscilloscope if available
- Compare expected vs actual timing

---

## Phase 3 — Sensor Driver

**Learning objectives:**
- Interface with I2C / SPI / analog sensors
- Understand the sensor's datasheet and communication protocol
- Write a basic polling driver
- Understand ADC for analog sensors

**What you need to do:**
- [ ] Read the TOF400C (VL53L1X) datasheet — understand its interface and registers
- [ ] Implement I2C/SPI communication to read sensor data
- [ ] Read the BMP280 (temperature + barometric pressure) (temperature + barometric pressure) datasheet
- [ ] Implement temperature reading (ADC or digital interface — [TBD])
- [ ] Print sensor readings over serial
- [ ] Validate sensor readings against known distances/temperatures

**Acceptance criteria:**
- [ ] TOF sensor returns distance readings that change when object distance changes
- [ ] Temperature sensor returns readings in a reasonable range
- [ ] Driver code is modular — separated from application logic
- [ ] Sensor datasheets annotated with your notes

**Recommended test methods:**
- Compare sensor output vs known measurement (ruler, thermometer)
- Serial output logging
- Logic analyzer on I2C/SPI bus to verify protocol correctness

---

## Phase 4 — CAN Basic Communication

**Learning objectives:**
- Understand CAN 2.0A/B frame format
- Configure STM32 CAN peripheral (bxCAN or FDCAN — depends on STM32 NUCLEO-F446RE)
- Transmit and receive a single CAN frame
- Use CAN transceiver hardware

**What you need to do:**
- [ ] Study CAN protocol basics: dominant/recessive, arbitration, CRC, ACK, error frames
- [ ] Wire WCMCU-20 to STM32 CAN TX/RX pins
- [ ] Configure CAN peripheral: bitrate [CAN_BITRATE], filters, interrupts
- [ ] Transmit a CAN frame and verify with logic analyzer
- [ ] Receive a CAN frame (loopback mode first, then with second node)
- [ ] Document CAN peripheral register configuration

**Acceptance criteria:**
- [ ] Can transmit a CAN frame visible on logic analyzer
- [ ] Can receive a CAN frame and read its data
- [ ] Understand bit timing configuration (prescaler, BS1, BS2, SJW)
- [ ] CAN error handling basics understood (bus-off, error passive)

**Recommended test methods:**
- Logic analyzer CAN decoder
- Loopback mode self-test
- Two-node communication test

---

## Phase 5 — Multi-node CAN

**Learning objectives:**
- Set up 2+ nodes on the same CAN bus
- Understand bus arbitration in practice
- Implement basic message routing

**What you need to do:**
- [ ] Wire two STM32 boards to the same CAN bus via transceivers
- [ ] Node A sends a message, Node B receives it
- [ ] Node B sends a response, Node A receives it
- [ ] Add a third node
- [ ] Verify arbitration behavior when multiple nodes transmit simultaneously
- [ ] Implement CAN acceptance filters to receive only relevant messages

**Acceptance criteria:**
- [ ] Three nodes communicate on the same bus without errors
- [ ] Each node only processes messages intended for it (filters work)
- [ ] Bus arbitration observed and understood
- [ ] No CAN error frames during normal operation

**Recommended test methods:**
- Logic analyzer showing multi-node traffic
- Error counter monitoring (TEC/REC)
- Deliberate collision test to observe arbitration

---

## Phase 6 — CAN Message Protocol

**Learning objectives:**
- Design a simple application-level CAN protocol
- Define message IDs, periods, and payloads
- Understand signal packing and byte ordering

**What you need to do:**
- [ ] Define your CAN message table (fill in the table in README.md)
- [ ] Decide on message IDs, priorities, and periods
- [ ] Define payload format for each message (which bytes mean what)
- [ ] Implement message encoding (pack sensor data into CAN payload)
- [ ] Implement message decoding (unpack CAN payload into application data)
- [ ] Document the protocol in `docs/can/CAN_PROTOCOL.md`

**Acceptance criteria:**
- [ ] CAN message table fully defined with IDs, periods, DLC, payload format
- [ ] Each node sends and receives its defined messages
- [ ] Data round-trips correctly (send value → receive → verify)
- [ ] Protocol document is complete and understandable

**Recommended test methods:**
- Send known sensor value → receive → compare
- Logic analyzer payload inspection
- Endianness and byte order verification

---

## Phase 7 — FreeRTOS

**Learning objectives:**
- Integrate FreeRTOS into the project
- Create tasks with appropriate priorities
- Understand task scheduling, context switching, stack sizing
- Use queues, semaphores, or mutexes for inter-task communication

**What you need to do:**
- [ ] Add FreeRTOS to one ECU node's project
- [ ] Create a task for periodic sensor reading
- [ ] Create a task for CAN message transmission
- [ ] Create a task for CAN message reception and processing
- [ ] Use a queue to pass data between tasks
- [ ] Configure task priorities based on timing requirements
- [ ] Measure actual task execution timing
- [ ] Handle stack overflow detection

**Acceptance criteria:**
- [ ] Multiple tasks run concurrently without starvation
- [ ] Inter-task communication works correctly (queue data is not corrupted)
- [ ] Task timing meets requirements (sensor read rate, CAN TX period)
- [ ] No stack overflows under normal operation
- [ ] Can explain your priority assignment rationale

**Recommended test methods:**
- FreeRTOS runtime stats (vTaskGetRunTimeStats)
- GPIO toggle in each task → logic analyzer to verify timing
- Stress test with high CAN bus load

---

## Phase 8 — ECU Application Architecture

**Learning objectives:**
- Design a clean software architecture for each ECU node
- Separate hardware abstraction, drivers, application logic, and communication
- Implement the full ECU behavior for each node

**What you need to do:**
- [ ] Define the software layers for each ECU node
- [ ] Implement [ECU_1_NAME] full application logic
- [ ] Implement [ECU_2_NAME] full application logic
- [ ] Implement [ECU_3_NAME] full application logic
- [ ] Ensure all three nodes work together as a system
- [ ] Document the architecture in `docs/architecture/`

**Acceptance criteria:**
- [ ] Each ECU performs its designated function
- [ ] All three ECUs communicate correctly over CAN
- [ ] Software layers are cleanly separated (driver vs app vs communication)
- [ ] System operates continuously without crashes for 1+ hour

**Recommended test methods:**
- End-to-end system test (all three nodes running)
- Long-duration soak test
- Boundary condition testing (sensor min/max values)

---

## Phase 9 — Fault Detection

**Learning objectives:**
- Implement basic fault detection and reporting
- Handle CAN bus errors gracefully
- Implement sensor fault detection (out-of-range, timeout)
- Understand watchdog timers

**What you need to do:**
- [ ] Implement sensor range checking (flag out-of-range readings)
- [ ] Implement CAN message timeout detection (expected message not received)
- [ ] Implement CAN bus error handling (bus-off recovery)
- [ ] Add a watchdog timer to detect task hangs
- [ ] Define fault codes and reporting mechanism over CAN
- [ ] Implement a simple fault log

**Acceptance criteria:**
- [ ] System detects and reports sensor faults
- [ ] System detects CAN message timeouts
- [ ] Watchdog resets the MCU if a task hangs
- [ ] Fault codes transmitted over CAN to other nodes
- [ ] System recovers gracefully from transient faults

**Recommended test methods:**
- Disconnect a sensor → verify fault detection
- Disconnect a CAN node → verify timeout detection
- Inject invalid sensor values → verify range checking
- Block a task with an infinite loop → verify watchdog reset

---

## Phase 10 — Testing / Debugging

**Learning objectives:**
- Systematically test the entire system
- Document test results
- Debug and fix discovered issues
- Learn to use debugging tools effectively

**What you need to do:**
- [ ] Execute all test cases in TEST_PLAN.md
- [ ] Record actual results and pass/fail status
- [ ] Debug any failures — root cause analysis
- [ ] Fix bugs and re-test
- [ ] Capture logic analyzer traces for key scenarios
- [ ] Performance profiling (CPU utilization, stack usage, CAN bus load)

**Acceptance criteria:**
- [ ] All test cases executed and documented
- [ ] All critical bugs fixed
- [ ] System passes a 1-hour soak test
- [ ] Performance metrics documented

**Recommended test methods:**
- Formal test execution per TEST_PLAN.md
- Logic analyzer captures archived in `docs/testing/`
- Memory usage analysis (stack high-water marks)

---

## Phase 11 — Documentation / Portfolio

**Learning objectives:**
- Write professional documentation for the project
- Create portfolio-quality presentation materials
- Reflect on what you learned

**What you need to do:**
- [ ] Update README.md with final information (replace all [TBD])
- [ ] Write a system architecture document with diagrams
- [ ] Write a CAN protocol specification
- [ ] Create a hardware wiring document with photos/diagrams
- [ ] Write a "Lessons Learned" document
- [ ] Take photos/videos of the system running
- [ ] Prepare a brief project summary for your resume/portfolio

**Acceptance criteria:**
- [ ] All documentation is complete and accurate
- [ ] README has no remaining [TBD] placeholders
- [ ] A new reader can understand the system from the docs alone
- [ ] Portfolio materials are professional and concise

**Recommended test methods:**
- Have someone else read the docs and see if they understand the system
- Review against job posting requirements for firmware/embedded roles
