# CAN Protocol Specification

<!-- TODO: Implement by Hongyi -->

## General

- **CAN Standard:** [TBD — CAN 2.0A / CAN 2.0B]
- **Bitrate:** [CAN_BITRATE]
- **Termination:** [TBD — 120 ohm resistors at each end of bus]

## Node List

| Node ID | Node Name    | Description      |
|---------|-------------|------------------|
| [TBD]   | [ECU_1_NAME] | [ECU_1_PURPOSE] |
| [TBD]   | [ECU_2_NAME] | [ECU_2_PURPOSE] |
| [TBD]   | [ECU_3_NAME] | [ECU_3_PURPOSE] |

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
