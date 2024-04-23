## 9 - 4.9

![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/f32c40f0-1a12-4fa1-bf69-bce238d5d79d)

a) What is the arithmetic intensity of this kernel? Justify your answer.
  
Arithmetic intensity is the ratio of floating-point operations to memory bytes accessed. In this example, there are 6 FP ops per loop (4 mul, 1 add, 1 sub) and 10 bytes of memory accessed. 
This makes it O(N) arithmetic intensity.

b) Convert this loop into VMIPS assembly code using strip mining.

LD R1, #300
LD R2, #64
loop:
LV V2, 0(Rare)
LV V3, 0(Rbre)
LV V4, 0(Raim)
LV V5, 0(Rbim)
MULVV V6, V2, V3
MULVV V7, V4, V5
MULVV V8, V2, V5
MULVV V9, V4, V3
SUBVV V10, V6, V7
ADDVV V11, V8, V9
SV V10, 0(Rcre)
SV V11, 0(Rcim)
DADDIU Rare, Rare, #64
DADDIU Rbre, Rbre, #64
DADDIU Raim, Raim, #64
DADDIU Rbim, Rbim, #64
DADDIU Rcre, Rcre, #64
DADDIU Rcim, Rcim, #64
DSUBIU R1,R1,#64
BGT R1, R2, loop
; last couple cases handle with scalar instructions

c) Assuming chaining and a single memory pipeline, how many chimes are required? How many clock cycles are required per complex result value, including start-up overhead?

A chime is completing a vector operation. There are 8 loads, 4 muls, 1 sub, 1 add, 2 stores. So 16 chimes.

All of the units need to start up, so 15+8+5 startup cycles. Chaining allows operations to start before previous one is finished. If we say that a new operation is 5 cycles, and each additional one is 1 more cycle,
we will get 5 + 7*1 + 4+1+1+2 = 20 cycles + 28 startup cycles, or 48 total.

## 8 - 1.9
You are designing a system for a real-time application in which
specific deadlines must be met. Finishing the computation faster gains nothing.
You find that your system can execute the necessary code, in the worst case,
twice as fast as necessary.
- a: How much energy do you save if you execute at the current speed
and turn off the system when the computation is complete?

Energy is proportional to voltage squared. Also, if you want to execute at current speed, it is 50% less than twice as fast, requiring 50% less voltage.

Energy_new/Energy_old = (voltage*0.5)^2/(voltage)  = 0.25

Since the process is running 2 times longer, multiply the ratio by 2. So you save 50% energy.

- b: How much energy do you save if you set the voltage and frequency to be half as much?

Same as a?

## 7 - 3.14
In this exercise, we look at how software techniques
can extract instruction-level parallelism (ILP) in a common vector loop. The following loop is the so-called DAXPY loop (double-precision aX plus Y) and
is the central operation in Gaussian elimination. The following code implements the DAXPY operation, Y = aX + Y, for a vector length 100. Initially, R1 is
set to the base address of array X and R2 is set to the base address of Y:

![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/11fa04f4-161c-49bf-88a1-1191587983f8)

Assume the functional unit latencies as shown in the table below. Assume a onecycle delayed branch that resolves in the ID stage. Assume that results are fully
bypassed. 

![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/457862ff-e313-4c3d-af89-5c29c96a4d4f)

a.

Solution doesn't make sense.

Unscheduled:
- L.D    F2,0(R1)
- Stall
- Stall
- MUL.D  F4,F2,F0
- Stall
- Stall
- Stall
- Stall
- Stall
- Stall
- L.D    F6,0(R2)
- Stall
- Stall
- ADD.D  F6,F4,F6
- Stall
- Stall
- Stall
- Stall
- S.D    F6,0(R2)
- Stall
- Stall
- DADDIU R1,R1,#8
- Stall
- Stall
- DADDIU R2,R2,#8
- Stall
- Stall
- DSLTU  R3,R1,R4
- Stall
- Stall
- BNEZ R3,foo
- Stall

Scheduled:
- L.D    F2,0(R1)
- L.D    F6,0(R2)
- Stall
- MUL.D  F4,F2,F0
- DADDIU R1,R1,#8
- DADDIU R2,R2,#8
- DSLTU  R3,R1,R4
- Stall
- Stall
- Stall
- ADD.D  F6,F4,F6
- Stall
- Stall
- Stall
- Stall
- S.D    F6,0(R2)
- Stall
- Stall
- BNEZ R3,foo
- Stall

Execution time:
- Unscheduled: 32 cycles
- Scheduled: 20 cycles

The clock would have to be 32/20 = 1.6, or 60% faster to match the performance improvement achieved by scheduling.

b.

Solution doesn't make sense.

The loop must be unrolled 7 times because there are 6 stall slots after MUL.D

Instruction schedule:
- L.D    F2,0(R1)
- L.D    F8,8(R1)
- L.D    F14,16(R1)
- L.D    F20,24(R1)
- L.D    F26,32(R1)
- L.D    F32,40(R1)
- L.D    F38,48(R1)
- MUL.D  F4,F2,F0
- MUL.D  F10,F8,F0
- MUL.D  F16,F14,F0
- MUL.D  F22,F20,F0
- MUL.D  F28,F26,F0
- MUL.D  F34,F32,F0
- MUL.D  F40,F38,F0
- L.D    F1,0(R2)
- L.D    F7,8(R2)
- L.D    F13,16(R2)
- L.D    F19,24(R2)
- L.D    F25,32(R2)
- L.D    F31,40(R2)
- L.D    F37,48(R2)
- ADD.D  F1,F4,F1
- ADD.D  F7,F10,F7
- ADD.D  F13,F16,F13
- ADD.D  F19,F22,F19
- ADD.D  F25,F28,F25
- ADD.D  F31,F34,F31
- ADD.D  F37,F40,F37
- S.D    F1,0(R2)
- S.D    F7,8(R2)
- DADDIU R1,R1,#56
- S.D    F13,16(R2)
- S.D    F19,24(R2)
- DADDIU R2,R2,#56
- S.D    F25,32(R2)
- S.D    F31,40(R2)
- DSLTU  R3,R1,R4
- S.D    F37,48(R2)
- BNEZ R3,foo

39 cycles for 7 elements, so 5.57 cycles per element

c.

Solution doesn't make sense.

Mem ref 1 | Mem ref 2 | FP op 1 | FP op 2 | Integer op/branch
 --- | --- | --- | --- | ---
 L.D    F2,0(R1)   | L.D    F20,24(R1) |                    |                    |
 L.D    F8,8(R1)   | L.D    F26,32(R1) |                    |                    |
 L.D    F14,16(R1) | L.D    F32,40(R1) | MUL.D  F4,F2,F0    | MUL.D  F22,F20,F0  |
 L.D    F1,0(R2)   | L.D    F19,24(R2) | MUL.D  F10,F8,F0   | MUL.D  F28,F26,F0  |
 L.D    F7,8(R2)   | L.D    F25,32(R2) | MUL.D  F16,F14,F0  | MUL.D  F34,F32,F0  |
 L.D    F13,16(R2) | L.D    F31,40(R2) | ADD.D  F1,F4,F1    | ADD.D  F19,F22,F19 |
   .               |  .                | ADD.D  F7,F10,F7   | ADD.D  F25,F28,F25 |
   .               |  .                | ADD.D  F13,F16,F13 | ADD.D  F31,F34,F31 | DADDIU R1,R1,#48
 S.D    F1,0(R2)   | S.D    F19,24(R2) |                    |                    | DADDIU R2,R2,#48
 S.D    F7,8(R2)   | S.D    F25,32(R2) |                    |                    | DSLTU  R3,R1,R4
 S.D    F13,16(R2) | S.D    F31,40(R2) |                    |                    | BNEZ R3,foo

 Does 6 elements in 11 cycles, so execution time per element is 1.83 cycles per element. 34 out of 55 cycles are used. It uses 19 registers.

## 6 - 2.13
![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/8fcbb4f4-0c9f-4a67-93a0-1bc519b5d56b)

![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/b83a711c-da84-41b5-835c-b1837fd88f35)

- a: A 2GB DRAM with parity or ECC effectively has 9 bit bytes. This would require 18 1Gb DRAMs. To create 72 output bits, each one would need to output 72/18 = 4 bits.
- b: If there are 8B for data, we need burst length of 32B/8B = 4
- c: DDR2-667 is 667 * 8 = 5336 MB/sec and DDR2-533 is 533 * 8 = 4264 MB/sec

## 5 - https://www.cs.utexas.edu/~lorenzo/corsi/cs372/06F/hw/3sol.html

1 - ![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/b6f3dd23-de2d-490b-b1ab-bdb4b4d9bc1b)

1.1 - What is the page size in such a system?

There are 8 bits for offset, so page size is 2^8B.

1.2 - What is the size of a page table for a process that has 256K of memory starting at address 0?

256KB = 2^18B, or 2^10 pages

Each L3 table address 2^6 pages, so we need 2^4 L3 tables.

Each L2 table addresses 2^8 L3 tables, so we only need 2^4 entries in 1 L2 table.

Since we only need 1 L2 table, we need 1 entry in the L1 table.

Assuming 2B per entry, we need 1 * 2^10 * 2 + 1 * 2^8 * 2 + 2^4 * 2^6* 2 = 4608B.

2 - A computer system has a 36-bit virtual address space with a page size of 8K, and 4 bytes per page table entry.

2.1 - How many pages are in the virtual address space?

2^36 / 2^13 = 2^23 pages

2.2 - What is the maximum size of addressable physical memory in this system?

We can reference 2^32 pages with 36 bit virtual address and 4B per PTE. Thus, max size of physical addressable memory is 2^32 * 2^13 = 2^45

2.3 - If the average process size is 8GB, would you use a one-level, two-level, or three-level page table? Why?

8GB = 2^33B, Page size = 2^13B

One-level:

If we have one table, it would have 2^23 PTE. If each of them is 4B, we would need 2^25, or 32MB of memory.

Two-level:

V.A is 36 bits, offset is 13 bits. Thus, 23 bits remaining: 11 for L2 tables, 12 for L1 table.

We need 2^33/2^13 = 2^20 pages => 2^20/2^11 = 2^9 L2 tables

1 * 2^12 * 4 + 2^9 * 2^11 * 4 = 2^13 + 2^22 = ~4MB

Three-level:

V.A split: 8,8,7,13

We need 2^20 pages => 2^20 / 2^7 = 2^13 L3 tables => 2^13/2^8 = 2^5 L2 tables => 2^5 entries in 1 L1 table

1 * 2^8 * 4 + 2^5 * 2^8 * 4 + 2^13 * 2^7 * 4 = ~4MB

It is better to pick level 2 because it has less overhead than leve 3, while using the same amount of memory.

3 - In a 32-bit machine we subdivide the virtual address into 4 pieces as follows:
8-bit    4-bit    8-bit    12-bit
We use a 3-level page table, such that the first 8 bits are for the first level and so on. Physical addresses are 44 bits and there are 4 protection bits per page.

3.1 - What is the page size in such a system?

Page size = 2^12B

3.2 - How much memory is consumed by the page table and wasted by internal fragmentation for a process that has 64K of memory starting at address 0?

64KB = 2^16B or 2^4 pages. Thus, we need 2^4 entries in 1 L3 table, 1 entry in 1 L2 table, 1 entry in 1 L1 table.

Page frame number = 2^44 / 2^12 = 2^32, adding 4 protection bits, each entry of level 3 would need to be 36 bits. Rounding to 64 bits, or 8B.

Memory consumed: 1 * 2^8 * 8 + 1 * 2^4 * 8 + 1 * 2^8 * 8 = 2048B

## 4 - 3.1
![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/ebc33478-ee37-4e16-bfdd-004f5ba9bfd6)
![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/7e911438-e85d-4224-a0dd-23b5986f4120)

If you cannot issue an instruction until the previous one is completed, there is no pipelining.

4+12+5+4+1+1+0+0+1+0+1+1(delay slot) = 30 cycles

Solution:

Each instruction takes 1 cycles to execute + extra latency cycles.

(1+4)+(1+12)+(1+5)+(1+4)+(1+1)+(1+1)+(1+0)+(1+0)+(1+1)+(1+0)+(1+1) = 40 cycles

## 3 - A.1
![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/79e01d59-65c1-412b-9c83-fcbeca14995f)
![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/6698e028-bcd0-4bbc-a915-4832f2dab383)

- load/store:
  - gap: 26.5 + 10.3 = 36.8%
  - gcc: 25.1 + 13.2 = 38.3%
  - average: (36.8+38.3)/2 = 37.55%
- alu instructions (add, sub, mul, compare, load imm, shift, AND, OR, XOR, other logical)
  - gap: 21.1 + 1.7 + 1.4 + 2.8 + 4.8 + 3.8 + 4.3 + 7.9 + 1.8 + 0.1 = 49.7%
  - gcc: 19.0 + 2.2 + 0.1 + 6.1 + 2.5 + 1.1 + 4.6 + 8.5 + 2.1 + 0.4 = 46.6%
  - average: 48.15%
- conditional branches:
  - gap: 9.3%
  - gcc: 12.1%
  - average: 10.7%
- jumps:
  - gap: 0.8%
  - gcc: 0.7%
  - average: 0.75%
 
CPI = 1.0 * 0.4815 + 1.4 * 0.3755 + 2.0 * 0.6 * 0.107 + 1.5 * 0.4 * 0.107 + 1.2 * 0.0075 = 0.4815 + 0.5257 + 0.1284 + 0.0642 + 0.009 = 1.2088

## 2 - 1.4
![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/974d3f7c-608b-4ea9-ab45-03aaa225bf7d)
![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/dd261a1e-45ab-4a76-8467-4602f1e459b4)

a.

Power needed = 66 + 2*2.3 + 7.9 = 78.5 W

Power supply is 80% efficient, so it should be 78.5/0.8 = 99 W

b.

power = 0.6*4 + 0.4*7.9 = 2.4 + 3.16 = 5.56

c.

seek7200 = 0.75 * seek5400

seek7200 + idle7200 = 100

seek5400 + idle5400 = 100

seek7200 * 7.9 + idle7200 * 4 = seek5400 * 7 + idle5400 * 2.9

idle5400 = 29.8%

## 1-H&P Ch 1.9
![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/240a927a-acc1-4307-a87e-8188a0dffcc5)

Speedup = (old execution time)/(new execution time)

(old execution time) = 1

(new execution time) = 0.6 + 0.4/10 (it will spend 10 times less time on the 40% part)

speedup = 1/0.64 = 1.56