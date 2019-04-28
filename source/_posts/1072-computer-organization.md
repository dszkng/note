---
title: 計算機組織
---

# 計算機組織

## 1. Computer Abstractions and Technology

### Abstractions architecture

![](2019-04-17-21-22-13.png)

ISA: Instruction set architecture (The hardware/software interface)

### Trade-off

#### Power Trends

$$
Power = Capacitive \, load \times Voltage^2 \times Frequency
$$

#### Integrated Circuit Cost

$$
Cost \, per \, die = \frac{ Cost \, per \, wafer }{ Dies \, per \, wafer * Yield}
$$

$$
Dies \, per \, wafer \approx \frac{Wafer \, area}{Die \, area}
$$

$$
Yield = \frac{1}{(1 + (Defects \, per \, area * Die \, area / 2))^2}
$$

### Performance

#### Relative Performance

$$
Performance = \frac{1}{Execution \, Time}
$$

e.g. X is n time faster than Y

$$
\frac{Performance_X}{Performance_Y} = \frac{Execution \, time_Y}{Execution \, time_X} = n
$$

#### Response Time and Throughput

* **Response time**(Elapsed time): The time it takes to do a task（**要完成一件工作，所需花費的時間**）
    * including all aspects (Processing, I/O, OS overhead, idle time)
    * Determines system performance
* **Throughput**: Total work done per unit time（**在一定時間內，所能完成的工作量**）
    * e.g., tasks/transactions/… per hour

#### CPU Clocking

![](2019-04-17-21-33-51.png)

* Clock period: duration of a clock cycle（一個 Cycle 花費的時間）
    * e.g., 250ps = 0.25ns = 250×10 –12 s
* Clock frequency (rate): cycles per second（一秒幾個 Cycle）
    * e.g., 4.0GHz = 4000MHz = 4.0×10 9 Hz

#### CPU time

**CPU處理給定的工作所花費的時間** (Discounts I/O time, other jobs' shares)

$$
CPU \, Time = Clock \, Cycles \times Clock \, Cycle \, Time = \frac{CPU \, Clock \, Cycles}{Clock \, Rate}
$$

公式理解：**處理工作時走了幾個 Cycle** 乘上**每走一個 Cycle 所要花費的時間**

* Performance improved by
    * Reducing number of clock cycles
    * Increasing clock rate
    * Hardware designer must often trade off clock rate against cycle count

#### Cycles per Instruction (CPI)

**每執行一個指令(Instruction)，所要花費的 Cycle**

$$
Clock \, Cycles = Instruction \, Count \times Cycles \, per \, Instruction
$$

* Instruction Count for a program
    * Determined by program, ISA and compiler
* Average cycles per instruction
    * Determined by CPU hardware

CPU Time 可以改寫為：

$$
CPU \, Time = Instruction \, Count \times CPI \times Clock \, Cycle \, Time = \frac{Instruction \, Count \times CPI}{Clock \, Rate}
$$

If different instruction classes take different numbers of cycles

$$
Clock \, Cycles = \sum_{i=1}^{n} (CPI_i \times Instruction \, Count_i)
$$

Weighted average CPI

$$
CPI = \frac{Clock \, Cycles}{Instruction \, Count} = \sum_{i=1}^{n}(\frac{CPI_i \times Instruction \, Count_i}{Instruction \, Count})
$$

#### Performance Summary

$$
CPI = \frac{Instructions}{Program} \times \frac{Clock \, cycles}{Instruction} \times \frac{Seconds}{Clock \, cycle}
$$

* Performance depends on
    * Algorithm: **affects IC**, possibly CPI
    * Programming language: **affects IC, CPI**
    * Compiler: **affects IC, CPI**
    * Instruction set architecture: **affects IC, CPI, Tc**

### [Amdahl's Law](https://en.wikipedia.org/wiki/Amdahl%27s_law)

Improving an aspect of a computer and expecting a proportional improvement in overall performance

$$
T_{improved} = \frac{T_{affected}}{Improvement \, factor} + T_{unaffected}
$$

### MIPS

Millions of Instructions Per Second

$$
MIPS = \frac{Instruction \, count}{Execution \, time \times 10^6} = \frac{Instruction \, count}{\frac{Instruction \, count \times CPI}{Clock \, rate} \times 10^6} = \frac{Clock rate}{CPI \times 10^6}
$$

> ps: MIPS無法拿來當作電腦效能的依據（**因為沒有考慮到 Instruction Count**）

## 2. Instructions: Language of the Computer

以下皆以 MIPS Instruction Set 為例子

### Design Principle

1. Simplicity favours regularity（MIPS Instruction 的 Format 都相近）
2. Smaller is faster（越少 Register 就越少電路）
3. Make the common case fast（e.g. addi vs add，減少 Load instruction)
4. Good design demands good compromises（根據需求有不同的format，但都是32bit且各個format盡量相似）

### Register Usage

| Register Number | Conventional Name | Usage
| --- | --- | --- 
| \$0 | \$zero | Hard-wired to 0
| \$1 | \$at | Reserved for pseudo-instructions
| \$2 - \$3 | \$v0, \$v1 | Return values from functions
| \$4 - \$7 | \$a0 - \$a3 | Arguments to functions - not preserved by subprograms
| **\$8 - \$15** | **\$t0 - \$t7** | Temporary data, not preserved by subprograms
| **\$16 - \$23** | **\$s0 - \$s7** | Saved registers, preserved by subprograms
| **\$24 - \$25** | **\$t8 - \$t9** | More temporary registers, not preserved by subprograms
| \$26 - \$27 | \$k0 - \$k1 | Reserved for kernel. Do not use.
| \$28 | \$gp | Global Area Pointer (base of global data segment)
| **\$29** | **\$sp** | Stack Pointer
| **\$30** | **\$fp** | Frame Pointer
| **\$31** | **\$ra** | Return Address
| \$f0 - \$f3 | - | Floating point return values
| \$f4 - \$f10 | - | Temporary registers, not preserved by subprograms
| \$f12 - \$f14 | - | First two arguments to subprograms, not preserved by subprograms
| \$f16 - \$f18 | - | More temporary registers, not preserved by subprograms
| \$f20 - \$f31 | - | Saved registers, preserved by subprograms

### Format

#### R-format (Register)

| op | rs | rt | rd | shamt | funct |
| --- | --- | --- | --- | --- | --- |
| 6 bits | 5 bits | 5 bits | 5 bits | 5 bits | 6 bits |

* op: operation code (opcode)
* rs: first source register number
* rt: second source register number
* rd: destination register number
* shamt: shift amount (00000 for now)
* funct: function code (extends opcode)

example) add \$t0, \$s1, \$s2

| op | rs | rt | rd | shamt | funct |
| --- | --- | --- | --- | --- | --- |
| special | $s1 | $s2 | $t0 | 0 | add |
| 0 | 17 | 18 | 8 | 0 | 32 |
| 000000 | 10001 | 10010 | 01000 | 00000 | 100000

$$
000000|10001|10010|01000|00000|100000_2 = 02324020_{16}
$$

#### I-format (Immediate)

| op | rs | rt | constant or address |
| --- | --- | --- | --- |
| 6 bits | 5 bits | 5 bits | 16 bits |

* Immediate arithmetic and load/store instructions
* rt: Target Register
* Constant: $-2^{15} ~ +2^{15} - 1$
* Address: offset added to base address in rs

example) addi \$t0, \$s1, -50

| op | rs | rt | constant or address |
| --- | --- | --- | --- |
| 8 | \$s1 | \$t0 | -50 |
| 8 | 17 | 8 | -50 |
| 001000 | 10001 | 01000 | 11111111 11001110 |

$$
001000|10001|01000|1111111111001110_2
$$

#### J-format (Jump)

| op | address |
| --- | --- |
| 6 bits | 26 bits |

### Addressing

#### Immediate Addressing

運算元是常數,且包裝在指令內部

![](2019-04-18-02-14-12.png)

#### Register Addressing

運算元是暫存器

![](2019-04-18-02-11-37.png)

#### Base Addressing

運算元存放在記憶體中,而位址本身是暫存器和指令中常數的和

![](2019-04-18-02-12-16.png)

#### PC-relative Addressing

位址是 PC 和指令中常數的加總

![](2019-04-18-02-13-43.png)

#### Pseduodirect Addressing

跳躍位址是指令的 26 位元再加上 PC 較高的位元

![](2019-04-18-02-13-19.png)

### Branch Addressing

PC: Program Counter

See: [ISA 2.4 MIPS: Addresses in branches and jumps](https://www.youtube.com/watch?v=1bP6alXjDrw)

#### Conditional Branch

根據比較結果改變程式流向，用 I-format (e.g. beq rs, rt, imm)

由於所有指令在記憶體中都是 4bytes 對齊的（一定都是 4 的倍數），最後 2 個 bit 沒有必要存，所以都是 0

![](2019-04-18-02-03-58.png)

#### Unconditional Branch

直接 jump 到指定的 Address 上

![](2019-04-18-02-05-12.png)

### Procedure Call

#### Leaf Procedure

See:

* [ISA 2.7 MIPS: Procedures and jal](https://www.youtube.com/watch?v=NjWZxz0GqFM)
* [ISA 2.8 MIPS Procedure call in detail (example)](https://www.youtube.com/watch?v=xPx5qdMaK9k)

#### Non-Leaf Procedure

Using stack to restore back registers

See:

* [ISA 2.9 MIPS: Saving and restoring registers to the stack](https://www.youtube.com/watch?v=y9Wv1RVbbNA)
* [ISA 2.10 Procedure Calls: Saving Registers (example 1)](https://www.youtube.com/watch?v=gbHagOGOfLc)
* [ISA 2.11 Procedure Calls: Saving registers (example 2)](https://www.youtube.com/watch?v=iST9dQolhHE)
* [ISA 2.12 Procedure call summary](https://www.youtube.com/watch?v=TYSYDrE4moE)

### Synchronization

兩個 procedure 對同一個 share variable 讀取寫入時，需要一個機制來協調兩者

See:

* [計算機組織 Chapter 2.10 Synchronization - Load link & Store conditional instructions - 朱宗賢老師](https://www.youtube.com/watch?v=QPpTBjf4Yrk)

## 3. Arithmetic for Computers

### Multiplication

* [12. Implementing Multiplication](https://www.youtube.com/watch?v=IeRm--sokdg)
* [12-1. Improving the Multiplication Hardware](https://www.youtube.com/watch?v=-QghzKWAZ6o)

#### Booth's Algorithm

[Booth's algorithm - Binary multiplication example | Computer Organization](https://www.youtube.com/watch?v=qVmZ-mZnYTI)

### Division

* [13. Implementing Division](https://www.youtube.com/watch?v=gNEm5QCe0eU)
* [13-1. Improving the Division Hardware](https://www.youtube.com/watch?v=7m6I7_3XdZ8)

### Floating Point

待補

#### IEEE 754

[Computer Science Concepts: IEEE 754 - Learn Freely](https://www.youtube.com/watch?v=oaf4v4wOuGI)

## Ch4

### p26

Store:

RegWrite - 0
ALUSrc - 1 (引進constant)
MemWrite - 1
MemRead - 0
MemtoReg - don't care

### p52

分 4 類
lw, sw, beq, r-type
由 second level gen 出 ALU signal
(讀 10 要再讀 funct field)

### p59

ALU_ctrl = ALUOp + Instruction[5:0]

# Buttom