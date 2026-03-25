# 4-Bit Breadboard Computer

## A Discrete Logic-Based CPU Built from Scratch

**Designed and built by:** Ezra Kyende (Tronic Zenergy)  
**Platform:** Breadboard Implementation using TTL/CMOS Logic ICs  
**Year:** 2025


##  Introduction
My name is Ezra, and this is one of many projects I have worked on. Before starting this 4-bit computer, I had already been learning fundamental computer concepts such as bus communication, ALU operations, and data movement between registers. This provided insight into how real CPUs are built and motivated the development of this project.

I believe that a deeper understanding of technology comes from learning the basics first and gradually building knowledge and skill. Although computers continue to evolve rapidly, understanding what happens inside these systems remains essential.

This project was implemented as a **4-bit computer built entirely on breadboards using discrete logic ICs**. The goal was to understand CPU operation at the hardware level, including registers, data buses, and arithmetic logic operations, **without the use of microcontrollers or software-based control**.


## System Overview

The 4-bit computer is a discrete logic-based system implemented entirely on breadboards. It follows a simple register-based architecture in which data is transferred through a shared 4-bit data bus and processed by a central Arithmetic Logic Unit (ALU). There is also a 4-bit bus that carries the instructions (select codes to ALU) and other operations.

The system is controlled by a clock signal that synchronizes data movement and operations across all modules. Control signals determine when registers load data, when devices place data onto the bus, and which operation the ALU performs (either arithmetic or logic operations).

The computer is designed for **educational and experimental purposes**, focusing on clarity of operation rather than performance or instruction density.

### Main Functional Blocks

| Block | Function |
|-------|----------|
| **RAM & ROM** | Holds the data and instructions |
| **4-Bit Data Bus** | Shared communication path for data |
| **Registers** | Store operands, instructions, and output data |
| **Arithmetic Logic Unit (ALU)** | Performs arithmetic and logic operations |
| **Control Logic** | Generates control signals for data flow and execution |
| **Input Interface** | Manual data and instruction entry |
| **Output Interface** | LED-based visualization of results and bus states (7-segment display) |


##  Core Architecture Details

### ROM
Initially I decided to use an M6294-04 ROM chip but later removed it after realizing it's a masked ROM. Since AT28C series ROM chips are not easy to find, I decided to use logic gates and counters to make a simple ROM that carries the startup sequence of the 4-bit computer. The sequence involved system checks on different units (control unit, ALU, and others) and resetting all registers and the program counter.

### RAM
The chip I used here is a **D436CX-15L Static RAM**. The chip has 8,192 memory locations, but I used only 15 of them since I was building a 4-bit computer. Later when designing an 8-bit computer, I will use more memory locations. Its function was to carry the program to be executed by the computer, where the program counter was used to address different memory locations inside the RAM to fetch instructions.

### Power Supply Unit
I used **2 × 3.7V, 5000mAh Li-ion batteries** as the power supply. From the batteries, I used an **L2596 buck converter** adjusted to give a constant 5V to the computer. On the power rails of the breadboards of different units, I used 0.1µF, 100µF, 10µF, and 1µF capacitors to reduce noise and increase system stability.

I also added a **blue LED bar 10-segment display** driven by an **LM3914** to display the battery level in real time.

### Address Bus
The address bus carried the address of the next instruction from RAM and ROM.

### Program Counter + Instruction Register
For the **Program Counter (PC)** I used a **CD4029 4-bit binary counter**. Its function was to give the address to RAM and ROM of the next instruction to be executed. The IC has jam inputs that allow presetting the counter to any state asynchronously with the clock. This feature enables loops and jumps.

For the **Instruction Register** I used a **74HC373 IC**, which is a high-speed, 8-bit, D-type transparent latch. I used it to store both the opcode and operand from RAM in a single fetch by the PC.

### Control Unit
Since I didn't have a ROM on the 4-bit computer, I had a hard task at the control unit to design it to give all the necessary control signals needed to execute various tasks. This meant having many logic gates, timing circuits, decoders, and encoders.

The control unit performed many operations like:
- Giving select codes for the ALU
- Control signals for Register A & B
- Controlling when the PC could fetch the next instruction from RAM

### System Bus
This is the communications path for the opcode and operand — **8 bits wide** (4 bits for opcode, 4 bits for operand).

### Registers A & B
These are the temporary storage units for data being manipulated. I used **2 × 74HC173 4-bit parallel load registers** with excellent input and output controls.

### ALU (Arithmetic and Logic Unit)
Here I used the **74LS181 4-bit ALU**, which supports many arithmetic operations (addition, subtraction) and logic operations (AND, OR, NOT, etc.).

### Output
For the output I used a **common cathode 7-segment display** driven by a **CD4511 (BCD to 7-segment) driver**. This was connected directly to the data bus.



## Challenges & Solutions

| Challenge | Description | Lesson Learned |
|-----------|-------------|----------------|
| **ROM** | Without a ROM chip, I had to wire how each instruction would be handled. This meant lots of logic gates, decoders, and encoders, leading to power instabilities, messy wiring, and extensive debugging. | A dedicated ROM simplifies control logic significantly. |
| **Data Transfer** | Registers A and B could not share data directly (instructions like MOV A, B weren't possible) because register outputs weren't connected to inputs. Each register needed its own bus transceiver. | Proper bus transceivers are essential for register-to-register transfers. |
| **ALU** | The ALU needed temporary storage for results. Using a shift register proved difficult due to clock errors and shifting issues. Also needed NOR gate logic to create a zero flag since the ALU only provides overflow and carry flags. | Dedicated accumulator register is better than shift registers for ALU output. |
| **MAR** | Without a Memory Address Register, the Program Counter had to multitask between providing addresses for RAM, instructions, and programmer circuits, making wiring and management difficult. | A dedicated MAR simplifies address management. |
| **Bus Transceiver** | Only used one transceiver at the ALU/register side. Other units like RAM and instruction register required careful timing control to prevent bus conflicts. | Every bus-connected device needs proper tri-state control. |
| **Output** | The BCD to 7-segment driver wasn't hexadecimal-capable, causing blank displays for values 10-15. | Hexadecimal display drivers are better for CPU projects. |
| **Breadboard Noise** | Significant noise caused errors during RAM programming and instruction execution. | Decoupling capacitors on every breadboard are essential. |


##  Instruction Set (Machine Code)

| Instruction | Opcode | Description |
|-------------|--------|-------------|
| MOV A | `1000` | Move data to Register A |
| MOV B | `1100` | Move data to Register B |
| STR A | `1001` | Store from Register A |
| STR B | `1010` | Store from Register B |
| ADD | `0100` | Addition operation |
| A + B (OR) | `0001` | Logical OR operation |
| NOT A | `0111` | Logical NOT operation |
| AB (AND) | `0110` | Logical AND operation |
| SUBTRACTION | `0011` | Subtraction operation |
| JO (Jump if Overflow) | `1101` | Conditional jump on overflow |
| HLT | `1111` | Halt execution |
| SLEEP (16s) | `10110000` | Sleep for 16 seconds |
| SLEEP (7s) | `10110100` | Sleep for 7 seconds |
| SLEEP (11s) | `10111000` | Sleep for 11 seconds |
| SLEEP (3.5s) | `10111100` | Sleep for 3.5 seconds |



##  Bill of Materials (Key Components)

| Component | Quantity | Purpose |
|-----------|----------|---------|
| D436CX-15L Static RAM | 1 | Program storage |
| CD4029 4-bit Counter | 1 | Program Counter |
| 74HC373 Latch | 1 | Instruction Register |
| 74HC173 Register | 2 | Registers A & B |
| 74LS181 ALU | 1 | Arithmetic/Logic operations |
| CD4511 BCD Driver | 1 | 7-segment display control |
| 74HC245 Transceiver | 1 | Bus control |
| LM2596 Buck Converter | 1 | Voltage regulation |
| LM3914 LED Driver | 1 | Battery level display |
| 3.7V 5000mAh Battery | 2 | Power source |
| 7-Segment Display | 1 | Output |
| Breadboards | Multiple | Platform |
| Various Logic Gates | Various | Control logic |
| Capacitors (0.1µF, 1µF, 10µF, 100µF) | Multiple | Noise filtering |



## Future Improvements

- [ ] Add a dedicated Memory Address Register (MAR)
- [ ] Implement hexadecimal 7-segment display driver
- [ ] Add proper bus transceivers for all bus-connected devices
- [ ] Replace control logic with a ROM-based microcode implementation
- [ ] Expand to 8-bit architecture
- [ ] Add more addressing modes
- [ ] Implement stack and subroutines
- [ ] Create an assembler for easier programming



*"Understanding technology comes from learning the basics first and gradually building knowledge and skill."*
