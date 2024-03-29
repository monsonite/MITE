# MITE


## A minimal 8-bit processor using a Bit Serial Architecture

![image](https://user-images.githubusercontent.com/758847/234409029-9489b60c-7d9f-40b0-8f49-5583a58ca1bd.png)

In a quest for a truly minimal computer, that can be built from readily available 74HC components on a low cost pcb, I have settled on a novel 8-bit, bit-serial architecture - a design that I call MITE.

MITE is the 8-bit version of the more complex 16-bit SPIDER. It uses far fewer ICs, and was designed as an Educational exercise in bit-serial architectures.

Any CPU will consist of several different building blocks: ALU, ROM, RAM, Control Unit, I/O.

MITE has all of these blocks, but mostly finds novel ways to reduce the hardware used in the ALU. Much of the rest of the system is fairly standard, based around parallel ROM and RAM and an 8-bit internal databus.

The bit serial ALU should be treated as a sub-system. Parallel Input data is serialised, processed one (pair) of bits at a time and the de-serialised for output to the rest of the parallel 8-bit memory bus system.

It is a hybrid system, with modern parallel addresses memory, but using a very minimal hardware bit serial ALU.

## Bit-Serial ALU

MITE is based around a pair of 8-bit shift registers A (Accumulator) and B (Bus). These shift registers feed the data into the bit-serial ALU, which in an earlier update, has been reduced from 8 TTL packages to a single 512 byte table, in a small ROM.

The discrete ALU features a full-adder capable of ADD and SUB, an inversion stage, for NEG and INV,  and a 2 input multiplexer to select between AND, XOR and OR boolean functions.

This can be reduced to just 4 quad 74HCxx packages  - plus a flip-flop to retain any intermediate carry.

However co-ordinating this ALU needs a 3 chip clock and timing signal generator, plus additional chips to control the instruction, carry and other input signals.

So any bit-serial ALU carries an overhead of several control unit chips.

However a lot of the discrete logic devices can be replaced by a small ROM of 256 or 512 bytes.

## History

Whilst bit-serial architectures were widely used from the late 1940s until the 1970s, for calculators and mainframe computer (Cambridge EDSAC 1948) systems that used serial access memory, parallel access memory and VLSI has largely made these machines redundant and obscure to today's Engineers. 

This project is to explore the earlier bit-serial methods and gain a better overall understanding.

## Serial meets Parallel

Modern memory is all parallel access, so at some point we need to convert between the parallel memory domain and the bit-serial ALU domain. This is done with the two shift registers A and B. 

A is a 74HC164 serial to parallel shift register and B is a 74HC165 parallel to serial shift register. We will also use the 74HC595, which is serial to parallel but with a latching register and tristate data bus capabilities. The other device of interest is the 74HC299 - a universal bi-directional shift register that can convert 8-bit data to and from an 8-bit bus to the serial domain.

Back to the basic bit serial ALU, B accepts a parallel 8-bit word and converts it to serial, A accepts serial data and converts it back to a parallel byte. For simplicity this conversion process should only happen between the B and A registers, but that is not always easy to achieve. 


We also need addressing registers that can form memory addresses of up to 16-bits. These addresses are parallel buses, so we might need to form them by using additional serial to parallel shift registers. 

We also need to build in flexibility in addressing modes, so that programming becomes easier.


## System Overview.


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


## Circuit Description.


### Clock Sequencer and Timing Generator.

![image](https://user-images.githubusercontent.com/758847/234841533-0a16b8bf-5db3-4445-8212-b78e0b851e0e.png)


The heart of the circuit is the clock sequencer - a 74HC4017 decimal counter (U11) with 10 decoded outputs T0 to T9. These sequential pulses co-ordinate all timing operations of the CPU. Only T0,T1,T8 and T9 are used to control the system timing.

A S-R latch (U14) made from a 74HC02 quad 2-input NOR, is set and reset by by T1 and T9 is used to gate (GATE) the clock to provide a burst of 8 consecutive clock pulses. This pulse train GCLK is used to clock the 8-bit data through the various 8-bit shift registers. 

A hex inverter 74HC04 (U15) provides inverted versions of T0 and T8 used for other clocking and timing functions. Spare inverters in this package are used to form a crystal oscillator circuit (not shown)

### Principal Timing Pulses.

T0 Loads the B shift register with 8-bits of data from the data bus. If there is a Carry from the previous bit operation set this during timing state T0.

T1 Set the SR latch to gate 8 clock pulses to GCLK. 

T8 Use inverted T8 to clock the final transient contents of the Accumulator (Shift Register A) into the output register of a 74HC595, thus latching it for a whole timing sequence. Increments the PC or forces a Jump

T9 Reset the SR latch thus terminating the burst of 8 gated clock pulses, until the next T1.


![image](https://user-images.githubusercontent.com/758847/234836873-a4425002-45c9-4a9a-9964-720b61ee650a.png)


### Shift Registers.

MITE has two principal shift registers A (Accumulator) and B (Bus). The Accumulator (U1) is a 74HC164 serial to parallel shift register and the Bus register (U3) is a 74HC165 parallel to serial 8-bit shift register. Both of these registers are clocked by the gated clock train GCLK. The B register is loaded from data stored in the 64K x 16-bit ROM (U7) on the rising edge of /T0.


A LOAD instruction will take an 8-bit literal from the ROM, load it asynchronously into the B register, passing it a bit at a time, unmodified by the ALU,  so that it is loaded into the Accumulator register. This is how data is initialised into the Accumulator. The instruction $0000 is effectively a LOAD A, 00 which clears the previous contents of the Accumulator.

### Program Counter (PC) and Microcode Instruction ROM.

4-bit binary counters 74HC161 (U8 and U9) form an 8-bit presetttable Program Counter that is used to address the lower 8-bits of the 64K x 16-bit ROM. The PC is incremented at the end of T8 so that it points to the next instruction in ROM.

If a Jump instruction is executed, the Program Counter is loaded with the new value - taken from the value on the data bus.

The upper 8 address lines of the are used to select a specific 256 byte page within the Instruction ROM. These page addressses are used to implement bytecode primitive instructions - such as from the MINT language.


### Memory Address Register (MAR).

This consists of a pair of 74HC595 shift registers (U5 and U6). They can be loaded in sequence from the accumulator to form up to a 16-bit address for addressing the Static RAM. A further pair of devices (U16 and U17) form a serial half adder, so that the memory address register can be incremented at the end of each instruction cycle - so that it points to the next bytecode location in RAM.

This scheme for the MAR has subsequently been replaced with a pair of 74HC161 4-bit counters to address the low 8 address lines of the RAM and a 74HC573 register to address the upper (page) address of the RAM.


### Bit Serial ALU.

A bit serial ALU, as its name suggests processes two steams of bits supplied a pair at a time from the A and B shift registers. This serial approach is used to considerably reduce the amount of hardware required to perform ALU calculations, compared to a parallel ALU. 

The bit serial ALU contains a full-adder, a pair of XOR gates to allow the A and B operands to be selectively inverted and a 2-input multiplexer.

There is a little logic to allow the Carry input to be included at the start of the data stream, and some further logic to suppress the Carry when bitwise logical operations are being performed.


The ALU uses the trick that the SUM output of two input bits is the XOR function, and the Carry output of two input bits is the AND function.  So the half-adder will generate both A XOR B and A AND B. The two input multiplexer can choose either the XOR function or the AND function. Furthermore, the multiplexer will also generate the A OR B function. So with very few gates we get the basic AND, OR and XOR functions - almost for free from the the full-adder. As well as the logic functions we can get both Addition, ADD with Carry and Subtraction. The front end XOR gates allow A and B to be inverted  - which can provide Invert and Negate instructions.

![image](https://user-images.githubusercontent.com/758847/234596132-f546abca-367d-4fb8-9782-367d4565eeba.png)
 
It can be seen that a versatile bit serial ALU can be made with just 12 gates. This design served as a prototype, until the combinational logic above was assimilated into a small PROM (U12) as a functional look-up table. This PROM also incorporated the state logic for memory read and write operations and the jump logic.

It is necessary to capture any carry that may be generated during any addition or subtraction operations. This Carry bit is held in the D-type flipflop (U2) so that it can be added into the next bit-sum.


## List of ICs.

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

U12 28C16 2K x 8 EEPROM  BIT-SERIAL ALU

U13  74HC541 octal buffer

U14 74HC02 quad 2-input NOR for clock gating

U15 74HC04 hex inverter for CLOCK OSCILLATOR and GLUE LOGIC

U16

U17


## Instruction Set.

The key to any simple processor is the ingenuity of its instruction set and architecture ISA. A clever instruction set can make the most out of minimal hardware. Examples of this are the PDP-8, the RCA 1802, the MOS 6502, Steve Wozniak's SWEET-16 virtual 16-bit machine and the Gigatron TTL Computer by the late Marcel van Kervinck. All of these processors have had an influence on MITE.

A decision was made early on to use a 16-bit wide instruction word. This gives a lot more flexibility than an 8-bit word, and simplifies the instruction fetch cycle. It can be done very conveniently with a 16-bit x 64K AT27C1024 OTP ROM, although a pair of 28C256 EEPROM parts could alternatively be used during the development cycle.

The 16-bit instruction can conveniently be expressed as 4-bit or 8-bit hexadecimal fields. This simplifies the notation and makes the instructions more human readable, as well as making any assembler or C simulation easier to write. This concept came from the RCA 1802 and also Wozniak's Sweet-16.

The upper byte from ROM will be known as the instruction byte IR7:IR0.  The lower byte will be known as the data byte or payload byte D7:D0.

The Instruction byte is split, for convenience, into two 4-bit fields IR7:IR4 and IR3:IR0.  IR7:IR4 define the ALU operation and IR3:IR0 define the data source or the addressing mode. 

As stated above, the bit serial ALU can perform AND, OR, XOR, ADD, SUB operations on the contents of the Accumulator A, and the Bus Register B. It can also zero or invert A or B and provide negation (2s complement).


## UPDATE August 2023

Following a successful simulation using H.Neemann's "Digital" simulator, I have been working on an EagleCAD schematic and layout on a cheap 10x10cm pcb.

From the basic CPU - I wanted to extend it to include general purpose parallel I/O and also native SPI in hardware.

I have therefore made the following improvements:

1 An 8-bit parallel output port.
  
2 An 8-bit parallel input port - both have direct access to the parallel data bus.
 
3 An 8-bit serial to parallel output port - implemented using a 74HC595
 
4 A 74HC595 used to drive 8 LEDs - to show cpu data/status etc.
  
5 A 40 pin IDC connector that mimics the data and address and control lines of a 6502.

6 Reset switch, User button and USB connector for 5V supply.
 
7 A 74HCT139 for selecting data source to be put on bus. The spare half is used to select the I/O port and generate Slave Select signals (/SS) for external SPI devices.
 
8 RAM has been extended to 128Kx8. An 8-bit presettable counter is used to address the lower bits. An octal register is used to address the upper address lines. Address line A16 can be toggled  to access the unused 64K bytes.  The use of the counter for the lower addresses lines means that MINT source code can easily be stepped through, character by character. The upper address register  can be loaded to jump to a different User routine on a new page boundary.
 
The Instruction and data ROM can be accessed from any source of data on the bus. This could be text stored in the program RAM, or an external device such as an SPI source, a keyboard or any other source of ascii character data.

The ALU ROM has been extended to 64Kx16 bits. The will allow significantly more combinational control logic to be incorporated into this single device. 16 inputs and 16 outputs The original ALU ROM was just 256 bytes!

These changes, particulary adding sensible I/O and RAM addressing have added 5 chips to the original cpu "core" design, so it now uses 21 packages. However it massively increases the usefulness and usability of the system, effectively transforming it from a CPU to a full microcomputer.

As I suggested yesterday, this is effectively a Gigatron - a modified Harvard architecture, but implements the ALU as a bit serial design in a single ROM, rather than 8-bit parallel ALU that used 8, 74HC153 multiplexer chips and two 74HC283 adders. Additionaly, much of the Control Unit logic, 6 chips and and a diode matrix of 30 diodes has been eliminated by putting the logic into the larger ROM.

The biggest takeaway from this project has been the challenges of getting a bit-serial ALU, to interface efficiently with a parallel memory system.  The bit-serial ALU could notionally be treated as a black box, which has a parallel to serial input interface, and a serial to parallel, output interface, with only these two connection points to what is otherwise a 8-bit parallel data bus.

The advantage of the bit-serial ALU, and ROM based combinational logic is a reduction from 38 IC packages to just 21, plus all the advantages of a much smaller pcb, enhanced I/O capabilities, lower overall costs and native SPI connectivity.(The Gigatron requires another 5 I/O packages and SPI bit banged in software).

The MITE could always be used as an I/O slave peripheral processor to the Gigatron.

Below is a screen shot of the latest pcb laayout.  Not shown are the internal VCC and GND planes - this is now a 4-layer board. Note that there are packages and other components underneath the wider ROM and ROM parts. As these parts will be socketed, there will be plenty room to stack them like this. The line of 8 diodes under the ROM is the Zero flag detector!

![image](https://github.com/monsonite/MITE/assets/758847/ae818401-7ffe-4449-8ee5-87532eead28c)

## Instruction Set - Update.

For various reasons this MITE design has taken on a very large influence from the Gigatron TTL Computer. The MITE could be considered to be a cousin of the Gigatron - but uses a bit-serial ALU for reduction in hardware complexity.

The Gigatron uses a 16-bit wide instruction ROM, which outputs an 8-bit wide instruction IR7:IR0 and an 8-bit data (or payload) word D7:D0.

There are 8 principle instructions defined by the opcode field IR7:IR5.

The ALU provides AND, OR, XOR, INV, ADD, SUB and NEG instructions.

The next field IR4:IR2 defines the addressing mode for memory referencing instructions. For jump or branch instructions it defines if there is a conditional attached to the Jump operation.

The remaining instruction field IR1:IR0 defines the source of the data that will be placed on the bus. 

There are 4 sources of data, A the Accumulator, D the Data word from the lower byte of the instruction word (a constant or an address), O the output from the currently addressed RAM address and I, an 8-bit input from a shift register - used to input external data from an external source such as a serial (PS/2) keyboard or a microcontroller - such as an Arduino.

This table summarises the Gigatron instruction set:

![image](https://github.com/monsonite/MITE/assets/758847/a7b039d4-28e8-4155-b1ea-5a55e5c5f358)

The SRAM is addressed by a pair of registers X and Y. 

X is formed from a pair of presettable 4-bit counters, driving memory address lines MA7:MA0 to allow incremental addressing (count) or jumping (load) within a 256 byte page. 

The Y register is an octal latch that addresses MA15:MA8. This allows Y to select a given 256 byte page of RAM and X to increment like a PC within the selected page.

There are other possible instruction set possibilities, but that of the Gigatron is concise and easily decoded in hardware.
  

## Hardware Reduction

The Gigatron uses 38 IC packages - and is primarily intended to generate colour graphics of 1/4 VGA resolution and sound for games playing.  The MITE is intended to be a more control application focussed system with a wide range of GPIO and SPI implemented in native hardware.

However, by implementing the ALU as a bit serial device contained in a modest sized ROM - significant savings can be made to the hardware chip count - about a 30% reduction from 38 devices to 24 parts or fewer. This provides a major saving in component costs and pcb size.

## Hardware Capabilities.

MITE is best described as a cousin of the Gigatron that uses a bit-serial ALU in order to reduce package count - but at the trade-off af a reduced instruction rate.

MITE is intended for tasks that normally would be performed by an 8-bit microcontroller - such as a PIC, or AVR, but with the educational benefit that it is built from discrete ICs - and very few of them.

MITE provides both parallel and serial GPIO and SPI connectivity:

8-bit parallel input port

8-bit parallel output port

10 MHz SPI dual bidirectional port (MCP23S17).

Native SPI expansion using 74HC299 universal shift register

8 LEDs for status or data supply.

Industry standard SPI memory - Flash, SRAM or FRAM in 8 pin DIP


## UPDATE 15th September 2023

In an attempt to partition the bit serial ALU and it's supporting timing pulse generator from the parallel memory interface, creating specific parallel->serial-> parallel interfaces I have created a 32 pin "module" for the serial circuitry. It has an 8-bit parallel input and a separate 8-bit tristate parallel output.

It currently uses just 8 ICs, so is very compact.

The ALU is reduced to a small EPROM. It could be replaced with a much faster GAL22V10 with the limit to clock frequency being dictated by the various shift registers.

There is a 74HC4017 to generate the timing states T0 to T9. Of these only T0, T1, T8 and T9 are used to control the system. Timing states T0, T1, T8 and T9 are also brought out and can be used to coordinate ROM and RAM access.

A quad NOR used to gate the 8 clock pulses to form the gated clock burst GCLK.

A 74HC165 converts parallel data from the bus into serial bitstream B, and combines it with data from the Accumulator 74HC164 in the ALU.

Once the 8 bits are processed the result from the Accumulator is latched into a 74HC595 which has tristate parallel outputs back to the databus.

A zero detector outputs on the Z pin and any carry produced is output on the C pin.

There are about 36 flipflops in this design, and a 256 byte look-up table in the PROM. I guess that it would be a good contender for a small CPLD.

I am trying to see if I can eliminate the 74HC164 and just use the 74HC595 as the Accumulator - still working on that.


![image](https://github.com/monsonite/MITE/assets/758847/43e31319-2e15-4a1f-a020-dc377d11d213)

## More About the Instruction Set

With the generic bit serial ALU and timing generator reduced to just 6 chips, I can effectively park that problem and focus on other aspects of the architecture.

From my dealings with Gigatron, it became clear that a 16-bit wide instruction had significantly more clout than 8-bits, and allows at least 8 if not 12 bits to be used for addressing memory. 

The AT27C1024 is a low cost 64K x 16-bit OTP EPROM with a 45nS access time. Incidentally, the cost of reprogrammable EEPROM seems to have gone through the roof, post pandemic and still suffering from supply chain problems.

So I was brought up with the Z80, and I like the organisational structure that the A, BC, DE, HL  (etc) registers bring to the programming task. Whilst I am considerably less familiar with the 6502, I appreciate the benefits of the zero page RAM for frequently used variables and general scratch pad use.

It is likely that any code written for MITE, will be generated from TASM-32, or a custom Python script. We no longer have to commit to memory raw opcodes in hex, like we did 45 years ago. Modern tools allow so much faster progress.

So the primary goal of MITE, is to create a BSA machine capable of executing a bytecode language from RAM, or other data source.

John Hardy, Craig Jones and I wrote MINT a couple of years back, as a means of creating an extremely lightweight, human readable, interpreted Forth-like language. We used the Z80 as our guinea pig cpu, and the MINT interpreter was reduced to about 1700 lines of Z80 ASM. Whilst not fast, about 40,000 16-bit adds per second, it was a proof of concept, and a springboard to other ideas.

At the same time Chris Curl was working on a similar tiny language S4, after Sandor Schneider had introduced us to his bytecoded "STABLE" tiny-Forthlike language.

The next task I see in this project is to create an optimised CPU design, for the efficient execution of bytecoded languages.

From our experiences with MINT, John Hardy and I found that with a plethora of 16-bit registers, named a->z, that the traditional stack operations from Forth such as DUP, DROP, SWAP and OVER, had less of a role to play when programming in MINT. Instead the registers took a more primary function with temporary data storage. This is good news for the programmer - you no longer to do a whole lot of mental stack juggling to find something that is buried 4 or 6 layers down.

The ALU provides just 8 basic instructions.

LOAD
AND
OR
XOR
ADD
SUB
STORE
JUMP

These are coded into the bits IR6:IR4 of the instruction register.

IR7 is the indirection bit, if set it means use the contents of memory addressed by the remainder of the instruction register. If unset, it means use the 8-bit literal contained in the payload byte of the instruction.

The lower 4 bits IR3:IR0 are used to select one of 16 pseudo registers stored in RAM. These will be treated as 8-bit registers if IR0=0 or eight, 16-bit register pairs (BC, DE, HL etc) if IR0=1.








