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


It might be beneficial to consider the instruction ROM as holding a microcoded bytecode interpreter.


The RAM is a 62256 (32K x8) or 128K x8 part. Two further shift registers (74HC595) are combined to form a 16-bit Memory Addressing Register. 
There is a little bit of glue logic, a 74HC74 dual flip flop is used to hold the carry bit, a quad 2-input NOR and a hex inverter. A 74HC4017 provides the clock sequencer that generates the 8 pulse gated clock and other timing signals.


As it stands, MITE is implemented in just 15 packages, 12 are 74HC devices and there is the ROM, the RAM and the ALU-ROM.
To the outside world, MITE presents itself as an SPI Master device. This means that it can be used with a wide range of SPI peripherals including SPI memory (RAM, FRAM, Flash etc) and other devices such as I/O extenders, sensors, low cost displays (LED & KEY) and the like.


MITE takes 10 clock cycles to execute an 8-bit instruction. However it is anticipated that a 20MHz clock can be used, which will bring the performance to about 2 million instructions per second.


MITE is designed to execute bytecode languages in a mix of hardware and ROM based microcode. 
The next step is to get MITE transferred from the simulator, to EagleCAD and a low cost 10 x 10cm pcb.


