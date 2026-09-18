# System Architecture

## 1. System overview

The system is a staged embedded platform centered on a closed-loop
motor-control DUT (Device Under Test), connected through CAN to a
Linux-based debug, validation, and firmware-management host.

``` text
                    Linux host                         [planned]
        debug / validation / firmware manager
                         |
                     SocketCAN
                         |
                      USB-CAN
                         |
======================= CAN ==========================
            |                            |
      STM32 axis node              second axis/node    [planned]
            |
     +------+------+
     |             |
  TMC2209        AS5600
 STEP/DIR       position
     |          feedback
   NEMA-17
```

The architecture has two layers:

- **Layer 1 — Embedded control DUT:** one or more STM32 motor nodes
  running closed-loop position control, CAN communication, fault
  handling, and telemetry.
- **Layer 2 — Linux debug and validation tool:** a host application
  that observes, tests, updates, and validates the DUT through
  SocketCAN. Deliberately separated so it can eventually support
  hardware other than this motor controller.

------------------------------------------------------------------------

## 2. STM32 firmware architecture

### 2.1 Software layer model

``` text
Application
|
+-- Control
|   +-- position controller (1 kHz target)
|   +-- state machine
|   +-- safety logic
|
+-- Communication
|   +-- CAN RX/TX
|   +-- protocol parser
|   +-- heartbeat
|
+-- Diagnostics
|   +-- fault manager
|   +-- telemetry
|
+-- Drivers
|   +-- AS5600 (I2C, 12-bit absolute magnetic encoder)
|   +-- TMC2209 (STEP/DIR/EN; optional UART)
|   +-- CAN peripheral
|
+-- HAL / register access
```

The project begins with STM32Cube/HAL for controlled bring-up, then
selectively lowers important peripherals toward register-level
implementation to understand the hardware.

An RTOS is introduced only if task separation or timing requirements
justify it.

### 2.2 Motor node control path

``` text
target position (from CAN or local)
      |
      v
position controller
      |
      v
step command / timer
      |
      v
   TMC2209 (internal current chopping)
      |
      v
   NEMA-17
      |
      v
 mechanical axis
      |
      v
    AS5600 (absolute position)
      |
      +---------- feedback
```

The TMC2209 handles motor-current chopping internally. The STM32
focuses on:

- deterministic step generation (timer-based);
- absolute position measurement;
- position-loop control and error correction;
- state-machine design;
- timeout and fault handling;
- CAN communication;
- telemetry publication.

> **This is not FOC.** There is no Clarke/Park transform and no dq-axis
> current loop.

### 2.3 Node responsibilities

| Responsibility | Description |
|----------------|-------------|
| STEP generation | Timer-based, deterministic pulse output |
| DIR / ENABLE control | GPIO direction and enable signals |
| AS5600 acquisition | I2C read of absolute position |
| Position control | Closed-loop correction at target rate |
| State machine | Operational modes, transitions, safe states |
| CAN RX/TX | Command reception, telemetry transmission |
| Heartbeat | Commander timeout detection |
| Encoder-loss detection | Frozen/disconnected sensor handling |
| Stall / error handling | Excessive tracking error response |
| Telemetry publication | Periodic state broadcast over CAN |
| Safe-state behavior | Defined behavior on any fault |

------------------------------------------------------------------------

## 3. CAN physical network

Linear backbone topology with short node stubs:

``` text
       120 ohm                              120 ohm
CANH ----+====================================+----
         |                |                   |
      Node A           USB-CAN             Node B
         |                |                   |
CANL ----+====================================+----
```

Design rules:

- CANH and CANL are a twisted pair;
- exactly one 120 ohm terminator at each physical end;
- nodes attach in parallel;
- stubs kept short;
- shared reference ground between boards;
- with power removed, a correctly terminated bus measures ~60 ohm
  between CANH and CANL.

**Initial bitrate target: 500 kbit/s.**

------------------------------------------------------------------------

## 4. CAN protocol

The protocol remains small and explicit. All message definitions live in
the CAN protocol layer, not scattered through application code.

### 4.1 Message classes

**Commands:**

``` text
SET_POSITION
ENABLE_MOTOR
DISABLE_MOTOR
SET_CONTROL_GAIN
RESET_NODE
ENTER_BOOTLOADER
```

**Telemetry:**

``` text
POSITION
TARGET_POSITION
VELOCITY
CONTROL_ERROR
NODE_STATE
FAULT_FLAGS
FIRMWARE_VERSION
```

Optional telemetry may later include motor current, supply voltage,
temperature, or IMU data if hardware is added.

**Health and diagnostics:**

``` text
HEARTBEAT
FAULT_REPORT
NODE_STATUS
BOOT_STATUS
```

The exact CAN-ID allocation and payload contract will be frozen only
after the basic physical bus is validated.

------------------------------------------------------------------------

## 5. Fault model

Fault handling is part of the core project, not a cosmetic extension.

### 5.1 Communication faults

- Commander heartbeat timeout
- Missing or stale command
- Invalid command/state transition

### 5.2 Sensor faults

- Encoder disconnected
- Encoder reading frozen
- Implausible position change
- Excessive measurement noise

### 5.3 Control / mechanical faults

- Excessive tracking error
- Blocked axis
- Lost steps
- Actuator saturation

### 5.4 Electrical faults (where instrumentation supports it)

- Undervoltage
- Overcurrent
- Motor-driver fault

All faults should transition the controller into defined safe states and
generate enough telemetry for the Linux host to reconstruct what
happened.

------------------------------------------------------------------------

## 6. Bootloader and CAN firmware update — planned

### 6.1 Flash layout

``` text
STM32 Flash
+-----------------------+
| Bootloader            |
+-----------------------+
| Application           |
|                       |
+-----------------------+
```

### 6.2 Minimum boot path

``` text
reset
  |
bootloader
  |
validate application
  |
set application context (MSP, VTOR)
  |
jump to application
```

The first bootloader milestone is a real bootloader successfully jumping
to a relocated application on hardware — **not OTA**. CAN update mode
is added after the jump works.

### 6.3 CAN update V1

``` text
Linux host
    |
 UPDATE_START (target, size, CRC32, version)
    |
 firmware data + sequence numbers
    |
 UPDATE_END
    |
 CRC32 verification
    |
 reboot
    |
 firmware-version verification
```

V1 targets:

- Bootloader/application Flash separation
- Relocated application (linker origin + VTOR relocation)
- CAN firmware transfer with sequence numbers
- Flash erase/write
- CRC32 validation
- Application validity check
- Jump to application
- Firmware-version query

Deferred: A/B images, rollback, signed firmware, authenticated updates.

------------------------------------------------------------------------

## 7. Linux debug and validation tool — planned

### 7.1 SocketCAN path

``` text
Linux application
       |
   PF_CAN socket
       |
    SocketCAN
---------------------- kernel boundary
 Linux CAN subsystem
       |
   CAN interface
---------------------- hardware boundary
    USB-CAN adapter
       |
     CAN bus
       |
    STM32 DUT
```

The initial implementation is userspace software. Using SocketCAN does
**not** by itself constitute Linux kernel-driver development.

### 7.2 Planned components

``` text
Linux Embedded Debug Tool
|
+-- CAN interface
|   +-- SocketCAN
|
+-- Telemetry
|   +-- frame decoder
|   +-- logger
|   +-- rolling buffer (pre/post-fault capture)
|
+-- Test runner
|   +-- command execution
|   +-- assertions
|   +-- pass/fail evaluation
|
+-- Fault capture
|   +-- trigger detection
|   +-- pre-fault buffer
|   +-- post-fault capture
|
+-- Firmware manager
|   +-- image loader
|   +-- CAN transfer
|   +-- CRC / status handling
|   +-- version verification
|
+-- UI                                      [optional/later]
    +-- telemetry visualization
    +-- fault timeline
    +-- test control
```

### 7.3 Automated validation

The host should run repeatable physical tests:

``` text
TEST: position tracking

1. command 0 deg
2. wait for settling
3. command +30 deg
4. record telemetry
5. check maximum error
6. check settling time
7. verify no unexpected fault
8. command -30 deg
9. repeat
10. PASS / FAIL
```

Pre/post-fault telemetry buffer:

``` text
              fault
                |
----------------+---------------- time
       pre-fault | post-fault
          data   |    data
```

------------------------------------------------------------------------

## 8. Integrated workflow — long-term target

``` text
build firmware
      |
      v
flash / CAN update
      |
      v
verify firmware version
      |
      v
run automated test
      |
      v
exercise physical DUT
      |
      v
capture telemetry
      |
   +--+--+
   |     |
 PASS   FAULT
         |
         v
  preserve evidence
         |
         v
       debug
         |
         v
 modify firmware
         |
         +---------- repeat
```

The host is not merely a dashboard; it becomes part of the embedded
bring-up, debugging, and validation workflow.

------------------------------------------------------------------------

## 9. Hardware

### 9.1 Current / near-term

| Part | Role | Interface |
|------|------|-----------|
| NUCLEO-F446RE | Primary motor-control node | — |
| TMC2209 | Stepper driver | STEP / DIR / EN; UART optional |
| AS5600 | 12-bit absolute magnetic encoder | I2C |
| NEMA-17 | Test actuator | — |
| SN65HVD230 or equivalent | CAN physical-layer transceiver | CAN TX/RX <-> CANH/CANL |
| CAN wiring + 2x120 ohm | Physical bus | twisted pair |

### 9.2 Planned

| Part | Role |
|------|------|
| Second CAN-capable MCU node | Physical CAN peer / eventual second axis |
| USB-CAN adapter | Linux <-> CAN interface |
| Linux laptop or Raspberry Pi | Debug / validation / firmware-management host |
| Additional current/voltage sensing | Optional diagnostics and fault evidence |

An ESP32 may still be useful as a temporary CAN peer, UI/network node,
or test fixture, but Wi-Fi/MQTT/LCD are no longer the architectural
center of the project.

------------------------------------------------------------------------

## 10. Scope boundaries

This project is **not primarily**:

- an LVGL dashboard;
- an IoT/MQTT demo;
- a cloud application;
- an AI diagnosis project;
- a full FOC motor controller;
- a Linux kernel-driver project.

The immediate Linux goal is solid userspace systems programming and
integration with SocketCAN. A previously considered AI
root-cause-analysis layer is explicitly deferred.

------------------------------------------------------------------------

## 11. Engineering principles

1. Real hardware behavior over architecture diagrams
2. Measured timing over assumptions
3. Reproducible failures over anecdotal debugging
4. Explicit interfaces between layers
5. Observable state and useful telemetry
6. Safe and deterministic fault behavior
7. Small milestones that build and run on hardware
8. Reusable host tooling
9. Clear distinction between implemented and planned features
10. No technology added solely for resume keywords

> Does it make the embedded system more realistic, observable, testable,
> reliable, or reusable? If not, it is probably outside the current
> scope.
