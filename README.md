# Optical Encoder with BiSS-C Protocol

This project implements a Renishaw optical encoder interface using the BiSS-C protocol on an STM32F446RET6 microcontroller. It reads position data from a high-precision optical encoder and converts it into millimeter measurements.

## Overview

An optical encoder is a sensor that converts rotational or linear position into an electrical signal. This project specifically interfaces with a Renishaw encoder using the BiSS-C (Bidirectional Synchronous Serial) protocol, which is a digital interface commonly used in high-precision positioning systems.

## Hardware Requirements

- **Microcontroller**: STM32F446RET6 (ARM Cortex-M4)
- **Optical Encoder**: Renishaw encoder with BiSS-C interface
- **Development Board**: STM32 Nucleo-F446RE or compatible board
- **Connections**: 
  - Clock signal (MA) to PA5
  - Data signal (SLO) to PA6
  - UART2 for debugging (optional, TX: PA2, RX: PA3)

## Pin Configuration

| Pin   | Function     | Description                        |
|-------|--------------|----------------------------------- |
| PA5   | ENC_CLK      | Encoder clock output (Master)     |
| PA6   | ENC_DATA     | Encoder data input (Slave)        |
| PA2   | USART_TX     | UART transmit for debugging       |
| PA3   | USART_RX     | UART receive for debugging        |

## How It Works

### BiSS-C Protocol Implementation

The BiSS-C protocol is a master-slave serial communication protocol. The STM32 acts as the master and communicates with the Renishaw encoder (slave) using a bit-banging approach.

#### Communication Sequence:

1. **Idle State**: The clock line (MA) is held HIGH
2. **Acknowledge Search**: The master sends clock pulses until the encoder pulls the data line (SLO) LOW to acknowledge
3. **Start Bit Detection**: The master continues clocking until it detects the START bit (always HIGH)
4. **Zero Bit**: A mandatory "0" bit follows the START bit (skipped by the code)
5. **Data Reading**: 32 bits of position data are read MSB-first
6. **Timeout Period**: A mandatory 31.25µs wait before the next read cycle

### Position Calculation

The raw 32-bit value from the encoder is processed as follows:

```c
rawValue = raw_result >> 6;              // Right shift by 6 bits
position_MM = rawValue * 0.00005f;       // Convert to millimeters (50nm resolution)
myDistance = position_MM - start_MM;      // Calculate distance from start position
```

The encoder provides a resolution of 50 nanometers (0.00005 mm) per count.

## Features

- **Bit-Banging BiSS-C Protocol**: Software implementation of the BiSS-C serial protocol
- **High Precision**: 50nm (0.00005mm) position resolution
- **Error Handling**: Timeout detection if encoder doesn't respond
- **Relative Distance Measurement**: Tracks distance traveled from the starting position
- **Continuous Reading**: Main loop continuously polls the encoder at ~200Hz (5ms delay)

## Code Structure

### Main Functions

- `read_renishaw_position()`: Implements the BiSS-C protocol to read position data from the encoder
- `main()`: Initializes peripherals and continuously reads position in a loop
- `MX_GPIO_Init()`: Configures GPIO pins for encoder interface
- `MX_USART2_UART_Init()`: Initializes UART for potential debugging

### Key Variables

- `rawValue`: Raw position count from encoder
- `position_MM`: Absolute position in millimeters
- `start_MM`: Initial position when first measurement is taken
- `myDistance`: Relative distance from starting position

## Building and Flashing

This project was created using STM32CubeIDE and can be built using:

1. **STM32CubeIDE**: 
   - Open the project folder
   - Build the project (Project → Build All)
   - Flash to the board (Run → Debug or Run)

2. **Command Line**:
   ```bash
   # If you have arm-none-eabi-gcc toolchain installed
   make clean
   make all
   ```

## Usage

1. Connect the Renishaw optical encoder to the STM32 board according to the pin configuration
2. Power up the system
3. The firmware will:
   - Initialize the hardware
   - Read the initial position and set it as the reference point
   - Continuously read the encoder position
   - Calculate the distance traveled from the starting position

## Technical Specifications

- **Communication Protocol**: BiSS-C (Bidirectional Synchronous Serial)
- **Data Width**: 32 bits
- **Position Resolution**: 50 nanometers (0.00005 mm)
- **Sampling Rate**: ~200 Hz (5ms update interval)
- **MCU Clock**: 84 MHz (configured via PLL)
- **UART Baud Rate**: 115200 bps

## Understanding the BiSS-C Protocol

BiSS-C is a digital interface standard for position feedback systems:

- **Bidirectional**: Supports both reading sensor data and writing parameters
- **Synchronous**: Clock signal provided by the master (STM32)
- **Serial**: Data transmitted bit-by-bit over a single line
- **Deterministic**: Fixed timing requirements ensure predictable behavior

This implementation uses a simplified read-only approach optimized for position reading.

## Troubleshooting

- **0xFFFFFFFF Reading**: Indicates timeout - check encoder power and connections
- **Incorrect Measurements**: Verify the 0.00005 scale factor matches your encoder's resolution
- **Inconsistent Readings**: Ensure clock signal meets timing requirements (adjust NOP loops if needed)

## License

Copyright (c) 2026 STMicroelectronics. All rights reserved.

This software is provided AS-IS with no warranty.

## Future Improvements

- Add UART output for real-time position monitoring
- Implement CRC error checking for data integrity
- Support for reading encoder status and error bits
- Add support for velocity calculation
- Implement interrupt-based reading for better timing accuracy
