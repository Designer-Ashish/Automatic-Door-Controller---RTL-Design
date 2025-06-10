# Automatic Door Controller - RTL Design

## Overview

This repository contains a Register Transfer Level (RTL) design for an automatic door controller implemented in Verilog. The controller manages door operation based on motion detection and includes safety features to prevent closing when obstructions are present.

## Features

- Motion-activated door opening
- Configurable timeout period before closing
- Obstruction detection safety mechanism
- Finite State Machine (FSM) based control logic
- Edge detection for motion sensor inputs
- Synthesizable Verilog RTL code

## Block Diagram

```
                    +-------------------+
                    |  Motion Sensor    |
                    +---------+---------+
                              |
                    +---------v---------+
                    |   Sensor Interface |
                    +---------+---------+
                              |
                    +---------v---------+
                    |   Control Logic   |
                    +---------+---------+
                              |
                    +---------v---------+
                    |  Timer/Counter    |
                    +---------+---------+
                              |
                    +---------v---------+
                    |  Door Motor Driver|
                    +---------+---------+
                              |
                    +---------v---------+
                    |  Obstruction      |
                    |  Sensor          |
                    +-------------------+
```

## Files

- `automatic_door_controller.v` - Main RTL design file
- `automatic_door_tb.v` - Testbench for simulation
- `README.md` - This documentation file

## Parameters

The design includes the following configurable parameter:
- `TIMEOUT` - Sets the delay before door closing (default = 50,000,000 cycles @ 50MHz = 1 second)

## States

The controller implements a 4-state FSM:
1. **IDLE** - Door closed, waiting for motion
2. **OPENING** - Door is opening
3. **OPEN** - Door is open (waiting for timeout or new motion)
4. **CLOSING** - Door is closing (can be interrupted by motion or obstruction)

## Simulation

To run the testbench:
```bash
iverilog -o sim automatic_door_tb.v automatic_door_controller.v
vvp sim
gtkwave door_controller.vcd
```

## Synthesis

The design is synthesizable for FPGA or ASIC implementation. For synthesis:
1. Add to your project in your preferred EDA tool (Vivado, Quartus, etc.)
2. Set appropriate timing constraints
3. Map I/O to physical pins for your target hardware

## Future Enhancements

1. Add limit switch inputs for precise door position detection
2. Implement PWM control for smoother motor operation
3. Add energy-saving "partially open" state
4. Include time-of-day based operation modes
5. Add emergency override controls

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
