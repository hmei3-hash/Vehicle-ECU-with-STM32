# STM32 CAN Closed-Loop Motion Controller + Linux Validation Tool

A staged embedded-systems project built around a real closed-loop
motor-control DUT. The current implementation begins with one
**STM32F446RE** motor node driving a **TMC2209** stepper driver and
reading an **AS5600** magnetic encoder. The project then expands into a
multi-node CAN system and a Linux-based debugging, validation,
telemetry, and firmware-update toolchain.

> **Status:** Phase 1 bring-up is in progress. This README distinguishes
> current implementation from planned work. Planned architecture is not
> presented as completed functionality.

------------------------------------------------------------------------

## Project goal

The motor controller is not the final product by itself. It is the first
Device Under Test (DUT) for a broader embedded development workflow:

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

The project is intended to exercise four connected areas:

1.  deterministic STM32 firmware and peripheral bring-up;
2.  closed-loop motion control and fault handling;
3.  CAN-based distributed embedded systems;
4.  Linux-based host tooling for observation, testing, and firmware
    management.

The Linux tooling is deliberately separated from the DUT so it can
eventually support hardware other than this motor controller.

------------------------------------------------------------------------

## Current implementation

The repository currently targets a **NUCLEO-F446RE** motor node.

The STM32 side is organized into three layers:

``` text
application logic
       |
device drivers
       |
HAL / register access
```

Current repository scaffolding includes:

-   PlatformIO build environments;
-   STM32 clock-tree configuration;
-   LED/toolchain bring-up;
-   centralized application configuration;
-   AS5600 driver structure;
-   TMC2209 driver structure;
-   CAN protocol definitions;
-   host-side test and measurement directories.

Drivers are kept separate from application logic, and
transport-dependent drivers are designed so their transport can be
substituted during host-side testing.

------------------------------------------------------------------------

## Why closed-loop stepper control?

The project intentionally sits between an open-loop STEP/DIR demo and
full field-oriented control.

The **TMC2209** performs the motor-current chopping internally. The
STM32 therefore focuses on:

-   deterministic step generation;
-   absolute position measurement;
-   position-loop control;
-   CAN communication;
-   state-machine design;
-   timeout and fault handling;
-   measurement and validation.

The **AS5600** measures mechanical position so the controller can detect
and correct position error rather than assuming that every commanded
step was physically executed.

> **This is not FOC.** There is no Clarke/Park transform and no dq-axis
> current loop.

------------------------------------------------------------------------

## Layer 1 --- Embedded control DUT

### Motor node

The initial STM32 node owns the timing-critical control path:

``` text
target position
      |
      v
position controller
      |
      v
step command / timer
      |
      v
   TMC2209
      |
      v
   NEMA-17
      |
      v
 mechanical axis
      |
      v
    AS5600
      |
      +---------- feedback
```

Responsibilities:

-   timer-based STEP generation;
-   DIR / ENABLE control;
-   AS5600 acquisition;
-   position control;
-   state machine;
-   CAN RX/TX;
-   heartbeat and command timeout;
-   encoder-loss detection;
-   stall / excessive-error handling;
-   telemetry publication;
-   safe-state behavior.

The initial control target is a **1 kHz position loop**, subject to
measurement and adjustment during bring-up.

### Future multi-axis expansion

After the first axis is stable, the architecture may expand to a second
independent CAN motor node:

``` text
                 CAN backbone
          +----------+----------+
          |                     |
      Axis X node           Axis Y node
        STM32                 STM32
          |                     |
   motor + encoder       motor + encoder
```

Each axis should remain independently controllable and diagnosable. The
second node is a later phase, not a prerequisite for validating the
first motor node.

------------------------------------------------------------------------

## CAN physical network

The physical CAN network uses a linear backbone with short node stubs.

``` text
       120 ohm                              120 ohm
CANH ----+====================================+----
         |                |                   |
      Node A           USB-CAN             Node B
         |                |                   |
CANL ----+====================================+----
```

Design rules:

-   CANH and CANL are a twisted pair;
-   exactly one 120 ohm terminator is placed at each physical end;
-   nodes attach in parallel to the same bus;
-   stubs are kept short;
-   participating boards share a suitable reference ground;
-   with power removed, a correctly terminated two-end bus should
    measure approximately 60 ohm between CANH and CANL.

The initial CAN bitrate target is **500 kbit/s**.

------------------------------------------------------------------------

## CAN protocol

The protocol should remain small and explicit. Message definitions live
in the CAN protocol layer rather than being scattered through
application code.

Planned message classes include:

### Commands

``` text
SET_POSITION
ENABLE_MOTOR
DISABLE_MOTOR
SET_CONTROL_GAIN
RESET_NODE
ENTER_BOOTLOADER
```

### Telemetry

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
temperature, or IMU data if the hardware is added.

### Health and diagnostics

``` text
HEARTBEAT
FAULT_REPORT
NODE_STATUS
BOOT_STATUS
```

The exact CAN-ID allocation and payload contract will be frozen only
after the basic physical bus is validated.

------------------------------------------------------------------------

## Fault model

Fault handling is part of the core project rather than a cosmetic
extension.

Candidate reproducible faults include:

**Communication** - commander heartbeat timeout; - missing or stale
command; - invalid command/state transition.

**Sensor** - encoder disconnected; - encoder reading frozen; -
implausible position change; - excessive measurement noise.

**Control / mechanical** - excessive tracking error; - blocked axis; -
lost steps; - actuator saturation.

**Electrical** --- only where instrumentation supports it -
undervoltage; - overcurrent; - motor-driver fault.

Faults should transition the controller into defined states and generate
enough telemetry for the Linux host to reconstruct what happened.

------------------------------------------------------------------------

## Firmware architecture

The intended STM32 organization is:

``` text
Application
|
+-- Control
|   +-- position controller
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
|   +-- AS5600
|   +-- TMC2209
|   +-- CAN
|
+-- HAL / register access
```

The project begins with STM32Cube/HAL where useful for controlled
bring-up, then selectively lowers important peripherals toward
register-level implementation to understand the hardware rather than
merely replacing working code for stylistic reasons.

An RTOS is not required for the first implementation. It should only be
introduced if task separation or timing requirements justify it.

------------------------------------------------------------------------

## Bootloader and CAN firmware update --- planned

A later firmware phase adds a small STM32 bootloader and relocates the
application image.

``` text
STM32 Flash
+-----------------------+
| Bootloader            |
+-----------------------+
| Application           |
|                       |
+-----------------------+
```

Minimum boot path:

``` text
reset
  |
bootloader
  |
validate application
  |
set application context
  |
jump to application
```

The first bootloader milestone is **not OTA**. It is a real bootloader
successfully jumping to a relocated application on hardware.

After that works, CAN update mode can be added.

### CAN update V1

``` text
Linux host
    |
 UPDATE_START
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

-   bootloader/application Flash separation;
-   relocated application;
-   CAN firmware transfer;
-   sequence numbers;
-   Flash erase/write;
-   CRC32 validation;
-   application validity check;
-   jump to application;
-   firmware-version query.

Deferred reliability/security features include A/B images, rollback,
signed firmware, and authenticated updates.

------------------------------------------------------------------------

## Layer 2 --- Linux debug and validation tool --- planned

The Linux host communicates with the CAN bus through SocketCAN:

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
    USB-CAN
       |
     CAN bus
       |
    STM32 DUT
```

The initial Linux implementation is userspace software. Using SocketCAN
does **not** by itself constitute Linux kernel-driver development.

### Planned components

``` text
Linux Embedded Debug Tool
|
+-- CAN interface
|   +-- SocketCAN
|
+-- Telemetry
|   +-- frame decoder
|   +-- logger
|   +-- rolling buffer
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

The first version may be command-line based. A Qt/QML interface is
optional and should not block the underlying test/debug infrastructure.

------------------------------------------------------------------------

## Automated validation --- planned

The host should eventually run repeatable physical tests instead of
relying only on manual observation.

Example:

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

A rolling telemetry buffer should preserve data from before and after a
fault:

``` text
              fault
                |
----------------+---------------- time
       pre-fault | post-fault
          data   |    data
```

The initial target is a small set of high-quality, reproducible tests
rather than a large shallow test suite.

------------------------------------------------------------------------

## Integrated workflow --- long-term target

The layers are intended to form one development loop:

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

This is the main architectural reason for adding Linux tooling: the host
is not merely a dashboard; it becomes part of the embedded bring-up,
debugging, and validation workflow.

------------------------------------------------------------------------

## Development phases

  -----------------------------------------------------------------------
  Phase                   Deliverable             Status
  ----------------------- ----------------------- -----------------------
  1                       Toolchain, clock tree,  **in progress**
                          LED blink, clock        
                          verification            

  2                       Register-level lowering planned
                          of selected Phase 1     
                          GPIO/RCC concepts       

  3                       AS5600 over I2C; angle  planned
                          and magnet-health       
                          readout                 

  4                       TMC2209 STEP/DIR timer  planned
                          generation; open-loop   
                          motion                  

  5                       Position loop;          planned
                          closed-loop correction  
                          and characterization    

  6a                      STM32 CAN internal      planned
                          loopback: timing,       
                          filters, FIFO, ISR path 

  6b                      Physical CAN bus with a planned
                          real second CAN-capable 
                          node                    

  7                       Fault state machine:    planned
                          encoder loss,           
                          stall/error, commander  
                          timeout                 

  8                       Characterization: step  planned
                          response, recovery, bus 
                          load, timing            
                          measurements            

  9                       Bootloader + relocated  planned
                          application; real       
                          jump-to-app             
                          demonstration           

  10                      CAN firmware update     planned
                          with CRC and version    
                          verification            

  11                      Linux SocketCAN         planned
                          telemetry/logger        

  12                      Linux automated test    planned
                          runner + fault capture  

  13                      Integrated Linux        planned
                          firmware manager /      
                          validation workflow     

  14                      Second motor axis /     optional
                          broader DUT support     
  -----------------------------------------------------------------------

Phases are dependency-driven. Later architecture should not block
completion and measurement of earlier hardware milestones.

------------------------------------------------------------------------

## Immediate definition of done

The current priority is still Phase 1.

Phase 1 is complete only when the basic build/flash path and clock
configuration are demonstrated and measured on hardware. A blinking LED
alone is insufficient evidence of a correct PLL/timing configuration.

After that, implementation should progress toward real hardware
artifacts rather than additional architecture-only documentation:

``` text
clock verified
    ->
encoder works
    ->
motor moves
    ->
closed loop works
    ->
CAN loopback works
    ->
physical CAN works
    ->
fault behavior works
    ->
bootloader jumps to relocated APP
    ->
CAN update works
    ->
Linux tooling
```

------------------------------------------------------------------------

## Hardware

### Current / near-term

  -----------------------------------------------------------------------
  Part                    Role                    Interface
  ----------------------- ----------------------- -----------------------
  NUCLEO-F446RE           Primary motor-control   ---
                          node                    

  TMC2209                 Stepper driver          STEP / DIR / EN; UART
                                                  optional

  AS5600                  12-bit absolute         I2C
                          magnetic encoder        

  NEMA-17                 Test actuator           ---

  SN65HVD230 or           CAN physical-layer      CAN TX/RX <-> CANH/CANL
  equivalent              transceiver             

  CAN wiring + 2x120 ohm  Physical bus            twisted pair
  -----------------------------------------------------------------------

### Planned as the system expands

  -----------------------------------------------------------------------
  Part                                Role
  ----------------------------------- -----------------------------------
  Second CAN-capable MCU node         physical CAN peer / eventual second
                                      axis

  USB-CAN adapter                     Linux <-> CAN interface

  Linux laptop or Raspberry Pi        debug / validation /
                                      firmware-management host

  Additional current/voltage sensing  optional diagnostics and fault
                                      evidence
  -----------------------------------------------------------------------

An ESP32 may still be useful as a temporary CAN peer, UI/network node,
or test fixture, but Wi-Fi/MQTT/LCD are no longer the architectural
center of the project.

------------------------------------------------------------------------

## Repository layout

Current layout:

``` text
├── platformio.ini
├── include/
│   └── app_config.h
├── src/
│   └── main.c
├── lib/
│   ├── as5600/
│   ├── tmc2209/
│   └── can_proto/
├── test/
└── docs/
```

As later phases are implemented, bootloader and Linux-host code should
be separated clearly rather than mixed into the STM32 application tree.

A likely future organization is:

``` text
├── firmware/
│   ├── app/
│   └── bootloader/
├── linux/
│   ├── can/
│   ├── telemetry/
│   ├── tests/
│   └── firmware_manager/
├── docs/
└── hardware/
```

The repository should only be reorganized when implementation reaches
those phases; directory structure should follow working code rather than
precede it unnecessarily.

------------------------------------------------------------------------

## Scope boundaries

This project is **not primarily**:

-   an LVGL dashboard;
-   an IoT/MQTT demo;
-   a cloud application;
-   an AI diagnosis project;
-   a full FOC motor controller;
-   a Linux kernel-driver project.

Possible future Linux kernel/driver work is a separate extension. The
immediate Linux goal is solid userspace systems programming and
integration with SocketCAN.

A previously considered AI root-cause-analysis layer is explicitly
deferred. If it is ever revisited, it should consume structured
telemetry and test evidence produced by the existing system rather than
replacing the deterministic debugging infrastructure.

------------------------------------------------------------------------

## Engineering principles

Future changes should prioritize:

1.  real hardware behavior over architecture diagrams;
2.  measured timing over assumptions;
3.  reproducible failures over anecdotal debugging;
4.  explicit interfaces between layers;
5.  observable state and useful telemetry;
6.  safe and deterministic fault behavior;
7.  small milestones that build and run on hardware;
8.  reusable host tooling;
9.  clear distinction between implemented and planned features;
10. no technology added solely for resume keywords.

A useful test for every proposed feature is:

> Does it make the embedded system more realistic, observable, testable,
> reliable, or reusable?

If not, it is probably outside the current scope.

------------------------------------------------------------------------

## Build

Current STM32 environments:

``` bash
pio run -e node0
pio run -e canloop
pio run -e node0 -t upload
pio device monitor -b 115200
```

Environment names may change as the bootloader and additional nodes are
introduced.

------------------------------------------------------------------------

## Measurements and known issues

Measured results belong in `docs/measurements.md`.

Known failures should remain visible until they are reproduced,
understood, and resolved. Planned claims such as timing behavior, bus
load, control performance, or fault-recovery performance should not be
presented as measured results before the corresponding phase is
complete.

------------------------------------------------------------------------

**Author:** Hongyi Mei
