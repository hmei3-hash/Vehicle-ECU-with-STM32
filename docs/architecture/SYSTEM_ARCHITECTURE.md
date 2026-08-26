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
| Sensor Node   | Read TOF400C (VL53L1X) and BMP280 (temperature + barometric pressure) sensors, transmit data over CAN | TOF400C (I2C), BMP280 (temperature + barometric pressure) (I2C) | CAN TX (sensor data) |
| Control Node   | Receive sensor data via CAN, execute control logic, drive NEMA17 motor | CAN RX (sensor data) | NEMA17 with TMC2209 (STEP/DIR or UART), CAN TX (status) |
| Dashboard Node   | Receive data from all nodes via CAN, display system status (UART now, LCD later) | CAN RX (sensor data, control status) | UART serial print (future: LCD), Single LED status indicator |

## Data Flow

[TBD — describe how data flows between nodes]
