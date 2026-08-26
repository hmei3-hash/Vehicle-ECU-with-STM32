# CAN Protocol Specification

<!-- TODO: Implement by Hongyi -->

## General

- **CAN Standard:** [TBD — CAN 2.0A / CAN 2.0B]
- **Bitrate:** 250 kbps
- **Termination:** [TBD — 120 ohm resistors at each end of bus]

## Node List

| Node ID | Node Name    | Description      |
|---------|-------------|------------------|
| [TBD]   | Sensor Node | Read TOF400C (VL53L1X) and BMP280 (temperature + barometric pressure) sensors, transmit data over CAN |
| [TBD]   | Control Node | Receive sensor data via CAN, execute control logic, drive NEMA17 motor |
| [TBD]   | Dashboard Node | Receive data from all nodes via CAN, display system status (UART now, LCD later) |

## Message Definitions

| Message     | CAN ID   | Sender | Receiver | Period   | DLC   | Payload Format |
|-------------|----------|--------|----------|----------|-------|----------------|
| [MESSAGE_1] | [CAN_ID] | [NODE] | [NODE]   | [PERIOD] | [DLC] | [PAYLOAD]      |
| [MESSAGE_2] | [CAN_ID] | [NODE] | [NODE]   | [PERIOD] | [DLC] | [PAYLOAD]      |
| [MESSAGE_3] | [CAN_ID] | [NODE] | [NODE]   | [PERIOD] | [DLC] | [PAYLOAD]      |

## Payload Definitions

### [MESSAGE_1]

| Byte | Bit Range | Signal Name | Unit | Scale | Offset | Min | Max |
|------|-----------|-------------|------|-------|--------|-----|-----|
| [TBD] | [TBD]   | [TBD]       | [TBD]| [TBD] | [TBD]  | [TBD]| [TBD]|

## Error Handling

[TBD — define how CAN errors are handled]
