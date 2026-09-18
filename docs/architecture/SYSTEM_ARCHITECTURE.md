# CAN Gimbal Control System — Architecture Document

## 1. System Overview

A 2-axis embedded gimbal control system using STM32 nodes connected through CAN bus, together with a Linux-based debugging and validation tool.

```
                     Linux Debug Tool (Raspberry Pi)
                            |
       +--------------------+--------------------+
       |                    |                    |
  Test Runner        Telemetry Logger      Firmware Manager
       |                    |                    |
  Fault Injection     Fault Capture         firmware.bin
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

## 2. STM32 Boot Sequence

### 2.1 Hardware Boot

On power-on or reset, the CPU hardware automatically:

1. Resets all registers to default values
2. Reads the Vector Table from `0x00000000` (mapped to `0x08000000` via BOOT0 pin)
3. `Vector Table [+0x00]` -> loads as MSP (Main Stack Pointer)
4. `Vector Table [+0x04]` -> loads as PC (Program Counter), pointing to Reset_Handler

### 2.2 Reset_Handler (startup assembly)

1. Copy `.data` section from Flash to RAM (initialized globals)
2. Zero `.bss` section (uninitialized globals)
3. Call `SystemInit()` (configure system clock)
4. Call `main()`

### 2.3 Vector Table Structure

```
Offset      Content
+0x00      Initial MSP value
+0x04      Reset_Handler address
+0x08      NMI_Handler
+0x0C      HardFault_Handler
+0x10      MemManage_Handler
...
+0x5C      SysTick_Handler
+0x60      First peripheral interrupt
...
```

When an interrupt fires, the CPU looks up `SCB->VTOR` for the current Vector Table base, reads the handler address at the corresponding offset, and jumps to it.

## 3. Flash Partitioning

### 3.1 Layout

```
0x08000000  +---------------------------+
            |                           |
            |       Bootloader          |
            |      (32KB, 0x8000)       |
            |                           |
0x08008000  +---------------------------+
            |                           |
            |       Application         |
            |    (remaining flash)      |
            |                           |
            +---------------------------+
```

### 3.2 Application Linker Script

The application's linker script **must** set:

```
FLASH : ORIGIN = 0x08008000, LENGTH = 480K
```

If the origin is left at `0x08000000`, all computed addresses will be wrong and the CPU will HardFault on jump.

### 3.3 SCB->VTOR Relocation

On reset, `SCB->VTOR` defaults to `0x00000000` (Bootloader's Vector Table). After jumping to the application, interrupts would still route to the Bootloader's handlers — causing HardFault.

**The application's `main()` must set this as its first line:**

```c
SCB->VTOR = 0x08008000;
```

## 4. Bootloader -> Application Jump

### 4.1 Jump Code

```c
// In Bootloader main()

// 1. Disable all interrupts
__disable_irq();

// 2. Read APP's Vector Table
uint32_t app_msp   = *((uint32_t *)0x08008000);        // APP's MSP
uint32_t app_reset = *((uint32_t *)(0x08008000 + 4));   // APP's Reset_Handler

// 3. Set stack pointer
__set_MSP(app_msp);

// 4. Jump to APP
void (*jump)(void) = (void (*)(void))app_reset;
jump();
```

### 4.2 Complete Boot Flow (with Bootloader)

```
Hardware Boot
  |
  v
Read 0x08000000 -> Bootloader MSP -> load
Read 0x08000004 -> Bootloader Reset_Handler -> execute
  |
  v
Bootloader Reset_Handler
  |  copy .data, zero .bss, SystemInit()
  v
Bootloader main()
  |
  v
Need firmware update?
  Yes -> receive new firmware, write Flash, verify
Check APP valid?
  Yes -> disable IRQ, read APP MSP + Reset_Handler, jump
  No  -> wait for firmware update
  |
  v
APP Reset_Handler
  |  copy .data, zero .bss, SystemInit()
  v
APP main()
  |  SCB->VTOR = 0x08008000  (first line)
  v
Application running
```

## 5. CAN Node Definitions

| Node | Node ID | Role |
|------|---------|------|
| STM32 Control Node | 0x01 | Motor control + sensor acquisition |
| STM32 Control Node | 0x02 | Motor control + sensor acquisition |
| Linux Debug (via ESP32-S3 USB-CAN bridge) | 0x03 | Telemetry, testing, OTA firmware update |

## 6. CAN OTA Firmware Update Protocol V1

### 6.1 CAN ID Assignments

| Message | CAN ID | Direction | Purpose |
|---------|--------|-----------|---------|
| UPDATE_START | 0x100 | Linux -> STM32 | Begin update |
| UPDATE_DATA | 0x101 | Linux -> STM32 | Firmware data frame |
| UPDATE_END | 0x102 | Linux -> STM32 | Transfer complete, verify |
| UPDATE_ACK | 0x103 | STM32 -> Linux | Acknowledge |
| UPDATE_NACK | 0x104 | STM32 -> Linux | Reject / error |
| VERSION_REQUEST | 0x105 | Linux -> STM32 | Query firmware version |
| VERSION_RESPONSE | 0x106 | STM32 -> Linux | Reply version |
| REBOOT | 0x107 | Linux -> STM32 | Reboot command |

### 6.2 Frame Formats (8 bytes)

**UPDATE_START (0x100):**

```
[0] Target node ID (0x01 / 0x02)
[1] Firmware size low byte
[2] Firmware size high byte
[3] CRC32 byte 0
[4] CRC32 byte 1
[5] CRC32 byte 2
[6] CRC32 byte 3
[7] Firmware version number
```

**UPDATE_DATA (0x101):**

```
[0] Sequence number low byte
[1] Sequence number high byte
[2-7] 6 bytes of firmware data
```

**UPDATE_END (0x102):**

```
[0] Target node ID
[1-7] Reserved
```

**UPDATE_ACK (0x103):**

```
[0] Node ID (responder)
[1] Status (0x01 = success)
[2-7] Reserved
```

**UPDATE_NACK (0x104):**

```
[0] Node ID
[1] Error code:
      0x01 = Insufficient space
      0x02 = Sequence error
      0x03 = Flash write failure
      0x04 = CRC verification failed
[2-7] Reserved
```

**VERSION_REQUEST (0x105):**

```
[0] Target node ID
[1-7] Reserved
```

**VERSION_RESPONSE (0x106):**

```
[0] Node ID
[1] Version number
[2-7] Reserved
```

**REBOOT (0x107):**

```
[0] Target node ID
[1-7] Reserved
```

### 6.3 Three-Phase Update Flow

**Phase 1 — Handshake:**

```
Linux sends UPDATE_START (target node, firmware size, CRC32, version)
  |
  v
STM32 Bootloader receives
  |
  v
Check space available, node ID match
  |
  v
OK -> Erase APP flash region -> reply ACK
Not OK -> reply NACK (error 0x01)
```

**Phase 2 — Data Transfer:**

```
Linux chunks firmware.bin into 6-byte frames
  |
  v
Send UPDATE_DATA per frame (2-byte seq + 6-byte data)
  |
  v
STM32 per frame: check seq -> write Flash -> reply ACK
  |
  v
Seq error -> NACK (0x02)
Write fail -> NACK (0x03)
  |
  v
Linux receives ACK -> send next frame
Linux receives NACK or 500ms timeout -> retry (max 3x)
3 retries failed -> abort update
```

**Phase 3 — Verification:**

```
Linux sends UPDATE_END
  |
  v
STM32 computes CRC32 of received data
  |
  v
Match -> ACK -> Linux sends REBOOT -> STM32 jumps to APP
Mismatch -> NACK (0x04) -> APP region marked invalid, return to IDLE
```

### 6.4 Timeout & Retry Parameters

| Parameter | Value |
|-----------|-------|
| ACK wait timeout | 500ms |
| Max retries per frame | 3 |
| Handshake timeout | 2000ms |

### 6.5 Bootloader State Machine

```
             +--------------------------------------+
             |                                      |
             v                                      |
           IDLE                                     |
             |                                      |
        Receive UPDATE_START                        |
             |                                      |
             v                                      |
          ERASING (erase APP flash region)          |
             |                                      |
          Erase done, reply ACK                     |
             |                                      |
             v                                      |
        RECEIVING <------+                          |
             |           |                          |
        Receive DATA     |                          |
        Write Flash, ACK-+  (loop)                  |
             |                                      |
        Receive UPDATE_END                          |
             |                                      |
             v                                      |
         VERIFYING (compute CRC32)                  |
             |                                      |
        +----+----+                                 |
      Pass      Fail                                |
        |         |                                 |
        v         v                                 |
   COMPLETE     ERROR ---------back to IDLE---------+
        |
   Receive REBOOT
        |
        v
   Jump to APP
```

Any error or timeout returns to IDLE. The Bootloader never crashes.

## 7. Hardware Architecture

### 7.1 Physical Connections

```
CAN Bus (CANH / CANL)
    |              |              |
 [SN65HVD230]  [SN65HVD230]  [SN65HVD230]
    |              |              |
  STM32          STM32        ESP32-S3
  Axis X         Axis Y      USB-CAN Bridge
 (0x01)         (0x02)           |
                                USB
                                 |
                           Raspberry Pi
                          (Debug + OTA)
                           (0x03)
```

### 7.2 Hardware List

| Hardware | Qty | Purpose |
|----------|-----|---------|
| STM32 NUCLEO-F446RE | 2 | Motor control + sensor nodes |
| ESP32-S3 | 1 | USB-CAN bridge (SLCAN firmware) |
| SN65HVD230 | 3 | CAN transceiver (one per node) |
| Raspberry Pi | 1 | Linux upper computer |
| 120 Ohm termination resistor | 2 | CAN bus termination |

### 7.3 ESP32-S3 USB-CAN Bridge

The ESP32-S3 runs SLCAN firmware, connected to Raspberry Pi via USB:

```
Raspberry Pi
  |
USB (/dev/ttyACM0)
  |
ESP32-S3 (SLCAN: serial commands <-> TWAI CAN frames)
  |
SN65HVD230
  |
CAN Bus
```

Raspberry Pi setup:

```bash
# Map serial port to SocketCAN interface
sudo slcand -o -c -s6 /dev/ttyACM0 slcan0
sudo ip link set slcan0 up

# Monitor CAN bus
candump slcan0

# Send test frame
cansend slcan0 123#DEADBEEF
```

## 8. Linux Debug Tool Architecture

### 8.1 Structure

```
                Linux Debug Tool
                      |
     +----------------+----------------+
     |                |                |
Firmware Manager  Telemetry Logger  Test Runner
     |                |                |
     +----------------+----------------+
                      |
               CAN library (Python)
                      |
              socket(PF_CAN)
                      |
                   slcan0
                      |
                ESP32-S3 bridge
                      |
                  CAN Bus
```

### 8.2 Software Layer Model

```
+---------------------------+
| Application Layer         |  ECU-specific logic (control loop, sensor fusion)
+---------------------------+
| RTOS Layer                |  FreeRTOS tasks, queues, semaphores
+---------------------------+
| Communication Layer       |  CAN TX/RX, message encode/decode
+---------------------------+
| Driver Layer              |  GPIO, Timer, I2C, SPI, CAN peripheral
+---------------------------+
| Hardware Abstraction      |  STM32 HAL / register-level access
+---------------------------+
| Bootloader                |  Flash management, OTA, jump-to-APP
+---------------------------+
| Hardware                  |  STM32 NUCLEO-F446RE + peripherals
+---------------------------+
```

## 9. Development Workflow (Target)

```
Modify Firmware
       |
     Build
       |
  firmware.bin
       |
Linux Firmware Manager
       |
  CAN OTA Flash
       |
CRC Verification
       |
    Reboot
       |
Firmware Version Check
       |
Automated Test
       |
Telemetry Capture
       |
  PASS / FAIL
```

## 10. Scope

### V1 (Current Target)

- STM32 CAN communication
- ESP32-S3 SLCAN bridge
- Raspberry Pi SocketCAN path
- Bootloader + Flash erase/write
- CAN OTA end-to-end test

### V2 (Extensions)

- Batch ACK (faster OTA transfer)
- ESP8266 wireless OTA path
- Dashboard real-time telemetry display
- Telemetry Logger data recording
- Test Runner automated testing

### Out of Scope

- UDS / ISO-TP
- Secure Boot / cryptographic signing
- A/B firmware partitions
- Edge AI
- Cloud backend
