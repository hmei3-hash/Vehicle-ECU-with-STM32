# System Architecture

<!-- TODO: Implement by Hongyi -->
<!-- Draw or describe the overall system architecture here -->

## System Block Diagram

[TBD — draw or insert your system block diagram here]

## Software Layer Model

```
+---------------------------+
| Application Layer         |  ECU-specific logic
+---------------------------+
| RTOS Layer                |  FreeRTOS tasks, queues, semaphores
+---------------------------+
| Communication Layer       |  CAN TX/RX, message encode/decode
+---------------------------+
| Driver Layer              |  GPIO, ADC, Timer, I2C, SPI, CAN peripheral
+---------------------------+
| Hardware Abstraction      |  Register-level access
+---------------------------+
| Hardware                  |  STM32 NUCLEO-F446RE + peripherals
+---------------------------+
```

## ECU Node Responsibilities

| Node           | Role           | Inputs        | Outputs       |
|----------------|----------------|---------------|---------------|
| [ECU_1_NAME]   | [ECU_1_PURPOSE] | [ECU_1_INPUTS] | [ECU_1_OUTPUTS] |
| [ECU_2_NAME]   | [ECU_2_PURPOSE] | [ECU_2_INPUTS] | [ECU_2_OUTPUTS] |
| [ECU_3_NAME]   | [ECU_3_PURPOSE] | [ECU_3_INPUTS] | [ECU_3_OUTPUTS] |

## Data Flow

[TBD — describe how data flows between nodes]
