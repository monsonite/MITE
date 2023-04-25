# MITE
A minimal 8-bit processor using a Bit Serial Architecture

![image](https://user-images.githubusercontent.com/758847/234409029-9489b60c-7d9f-40b0-8f49-5583a58ca1bd.png)

In a quest for a truly minimal computer, that can be built from readily available 74HC components on a low cost pcb, I have settled on a novel 8-bit, bit-serial architecture - a design that I call MITE.

MITE is the 8-bit version of the more complex 16-bit SPIDER. It uses far fewer ICs, and was designed as a learning exercise in bit serial architectures.


MITE is based around a pair of 8-bit shift registers A (Accumulator) and B (Bus). These shift registers feed the data into the bit-serial ALU, which in an earlier update, has been reduced from 8 TTL packages to a single 512 byte table, in a small ROM.


The ALU provides AND, OR, XOR, INV, ADD, SUB and NEG instructions.


The instruction ROM is a 64K x 16-bit 27C1024. This has a program counter (2 x 74HC161) that drives the lower 8 address lines. The upper 8-address lines can be driven from the bus.


This simple approach means that bytecode instructions can be used to force the ROM to the start of a 256 byte page boundary. Whilst not particularly ROM efficient, it provides a very simple hardware mechanism to implement a bytecode interpreter.


Using 16-bit wide instruction ROM allows an 8-bit literal or jump address to be incorporated into the lower byte. This byte may not always be used, again leading to low code density - but it massively simplifies the instruction fetch process. All instructions are 16-bits wide.


There is half a plan to use the payload byte to implement PDP-8 style OPR instructions, which could be used to extend the usefulness of what is otherwise a somewhat spartan instruction sets. OPR instructions use the individual bit lines to control the hardware, when no access to memory is required.
With minimal hardware, you have to be rather creative to get the most from the architecture.


It might be beneficial to consider the instruction ROM as holding a microcoded bytecode interpreter.


The RAM is a 62256 (32K x8) or 128K x8 part. Two further shift registers (74HC595) are combined to form a 16-bit Memory Addressing Register.


There is a little bit of glue logic, a 74HC74 dual flip flop is used to hold the carry bit, a quad 2-input NOR and a hex inverter. A 74HC4017 provides the clock sequencer that generates the 8 pulse gated clock and other timing signals.


As it stands, MITE is implemented in just 15 packages, 12 are 74HC devices and there is the ROM, the RAM and the ALU-ROM.


There may be some further ICs required as the design concept matures, probably another dual flipflop to hold the zero and final carry flags.
And as explained in a subsequent post, the addressing of the RAM is somewhat clunky, and could do with further refinements.


What I have learnt from this concept, is just how much combinational logic can be reduced to a remarkably small table in ROM.
From a register perspective, there are approximately 60 flip-flops used in the design, primarily in the octal flip-flops and 4-bit counters. This would suggest that this design might be suitable for a CPLD with low macrocell count.


To the outside world, MITE presents itself as an SPI Master device. This means that it can be used with a wide range of SPI peripherals including SPI memory (RAM, FRAM, Flash etc) and other devices such as I/O extenders, sensors, low cost displays (LED & KEY) and the like.
MITE takes 10 clock cycles to execute an 8-bit instruction. However it is anticipated that a 20MHz clock can be used, which will bring the performance to about 2 million instructions per second.


MITE is designed to execute bytecode languages in a mix of hardware and ROM based microcode. 
The next step is to get MITE transferred from the simulator, to EagleCAD and a low cost 10x10cm pcb.


Circuit Description.

The heart of the circuit is the clock sequencer - a 74HC4017 decimal counter (U11) with 10 decoded outputs T0 to T9. These sequential pulses co-ordinate all timing operations of the CPU.

A S-R latch (U14) set and reset by by T1 and T9 is used to gate the clock to provide a burst of 8 consecutive clock pulses. This pulse train GCLK is used to clock the 8-bit data through the various 8-bit shift registers. Timing signals T0 and T8 are used for other time sequenced operations.


MITE has two principal shift registers A (Accumulator) and B (Bus). The Accumulator (U1) is a 74HC164 serial to parallel shift register and the Bus register (U3) is a 74HC165 parallel to serial 8-bit shift register. Both of these registers are clocked by the gated clock train GCLK. The B register is loaded from data stored in the 64K x 16-bit ROM (U7) on the rising edge of /T0.


A LOAD instruction will take an 8-bit literal from the ROM, load it asynchronously into the B register, passing it a bit at a time, unmodified by the ALU,  so that it is loaded into the Accumulator register. This is how data is initialised into the Accumulator. The instruction $0000 is effectively a LOAD A, 00 which clears the previous contents of the Accumulator.

PROGRAM COUNTER

4-bit binary counters 74HC161 (U8 and U9) form an 8-bit presetttable Program Counter that is used to address the lower 8-bits of the ROM. The PC is incremented at the end of T8 so that it points to the next instruction in ROM.


List of ICs.

U1 74HC164 8-bit serial to parallel shift register  ACCUMULATOR

U2 74HC74 dual D-type flipflop CARRY FF

U3 74HC165 8-bit parallel to serial shift register B REGISTER

U4 74HC595 8-bit serial to parallel 8-bit shift register (tristate)

U5 74HC595 8-bit serial to parallel 8-bit shift register (tristate) MEMORY ADDRESS LOW

U6 74HC595 8-bit serial to parallel 8-bit shift register (tristate) MEMORY ADDRESS HIGH

U7 27C1024 64K x 16-bit ROM

U8 74HC161 4-bit presettable counter   PROGRAM COUNTER

U9 74HC161 4-bit presettable counter   PROGRAM COUNTER

U10 62256 32K x 8 static RAM

U11 74HC4017 decoded decade counter CLOCK SEQUENCER

U12 28C16 2K x 8 EEPROM  SERIAL ALU

U13  74HC541 octal buffer

U14 74HC02 quad 2-input NOR

U15 74HC04 hex inverter CLOCK OSCILLATOR and GLUE LOGIC

U16

U17




