---
title: 計算機組織
---

# 計算機組織

## Computer Abstractions and Technology

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

## Instructions: Language of the Computer

以下皆以 MIPS Instruction Set 為例子

### Design Principle

1. Simplicity favours regularity（MIPS Instruction 的 Format 都相近）
2. Smaller is faster（越少 Register 就越少電路）
3. Make the common case fast（e.g. addi vs add，減少 Load instruction)
4. Good design demands good compromises（根據需求有不同的format，但都是32bit且各個format盡量相似）

### Register Usage

| Register Number | Conventional Name | Usage                                                            |
| --------------- | ----------------- | ---------------------------------------------------------------- |
| \$0             | \$zero            | Hard-wired to 0                                                  |
| \$1             | \$at              | Reserved for pseudo-instructions                                 |
| \$2 - \$3       | \$v0, \$v1        | Return values from functions                                     |
| \$4 - \$7       | \$a0 - \$a3       | Arguments to functions - not preserved by subprograms            |
| **\$8 - \$15**  | **\$t0 - \$t7**   | Temporary data, not preserved by subprograms                     |
| **\$16 - \$23** | **\$s0 - \$s7**   | Saved registers, preserved by subprograms                        |
| **\$24 - \$25** | **\$t8 - \$t9**   | More temporary registers, not preserved by subprograms           |
| \$26 - \$27     | \$k0 - \$k1       | Reserved for kernel. Do not use.                                 |
| \$28            | \$gp              | Global Area Pointer (base of global data segment)                |
| **\$29**        | **\$sp**          | Stack Pointer                                                    |
| **\$30**        | **\$fp**          | Frame Pointer                                                    |
| **\$31**        | **\$ra**          | Return Address                                                   |
| \$f0 - \$f3     | -                 | Floating point return values                                     |
| \$f4 - \$f10    | -                 | Temporary registers, not preserved by subprograms                |
| \$f12 - \$f14   | -                 | First two arguments to subprograms, not preserved by subprograms |
| \$f16 - \$f18   | -                 | More temporary registers, not preserved by subprograms           |
| \$f20 - \$f31   | -                 | Saved registers, preserved by subprograms                        |

### Format

#### R-format (Register)

| op     | rs     | rt     | rd     | shamt  | funct  |
| ------ | ------ | ------ | ------ | ------ | ------ |
| 6 bits | 5 bits | 5 bits | 5 bits | 5 bits | 6 bits |

* op: operation code (opcode)
* rs: first source register number
* rt: second source register number
* rd: destination register number
* shamt: shift amount (00000 for now)
* funct: function code (extends opcode)

example) add \$t0, \$s1, \$s2

| op      | rs    | rt    | rd    | shamt | funct  |
| ------- | ----- | ----- | ----- | ----- | ------ |
| special | $s1   | $s2   | $t0   | 0     | add    |
| 0       | 17    | 18    | 8     | 0     | 32     |
| 000000  | 10001 | 10010 | 01000 | 00000 | 100000 |

$$
000000|10001|10010|01000|00000|100000_2 = 02324020_{16}
$$

#### I-format (Immediate)

| op     | rs     | rt     | constant or address |
| ------ | ------ | ------ | ------------------- |
| 6 bits | 5 bits | 5 bits | 16 bits             |

* Immediate arithmetic and load/store instructions
* rt: Target Register
* Constant: $-2^{15} ~ +2^{15} - 1$
* Address: offset added to base address in rs

example) addi \$t0, \$s1, -50

| op     | rs    | rt    | constant or address |
| ------ | ----- | ----- | ------------------- |
| 8      | \$s1  | \$t0  | -50                 |
| 8      | 17    | 8     | -50                 |
| 001000 | 10001 | 01000 | 11111111 11001110   |

$$
001000|10001|01000|1111111111001110_2
$$

#### J-format (Jump)

| op     | address |
| ------ | ------- |
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

## Arithmetic for Computers

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

## The Processor

### CPU Overview

![](2019-05-02-23-18-16.png)

因為不能把很多線接在一起，所以需要 MUX，還需要有 Control Unit 來控制 MUX：

![](2019-05-02-23-14-52.png)

### Instruction Fetch

![](2019-05-02-23-26-42.png)

* PC: 32-bits register
* 每次執行完指令後將 PC + 4 (1 個 instruction 4 bytes)

### Instructions Datapath

#### R-Format

| 6bits | 5bits | 5bits | 5bits | 5bits | 6bits |
| ----- | ----- | ----- | ----- | ----- | ----- |
| op    | rs    | rt    | rd    | shamt | funct |

![](2019-05-02-23-31-53.png)

* 讀 2 個 input: Read Register 1, 2
* 進 ALU 做運算
* 把算出來的結果存到 Write Register

#### I-Format

| 6bits | 5bits | 5bits | 16bits    |
| ----- | ----- | ----- | --------- |
| op    | rs    | rt    | immediate |

##### Load/Store

![](2019-05-02-23-34-01.png)

* Sign-extend (以 ALU 實做) 用作將 Address 做 16-bits 的 offset (I-type 的後 16-bits 要轉成 32-bits)
* Load: 讀 Memory 的資料然後更新 Register
* Store: 將 Register 的資料寫到 Memory

##### Branch

[Conditional Branch](#conditional-branch)

![](2019-05-02-23-51-53.png)

* 讀 2 個 Register
* 比較兩個 Register (用 ALU 的 Substract 還有 Check Zero output)
* 算 Target Address
  * Sign-extended
  * 左 shift 2-bits (因為一個 word 4-bits)
  * 加上 PC + 4

#### J-Format

| 6bits | 26bits         |
| ----- | -------------- |
| op    | target address |

[Unconditional branch](#unconditional-branch)

* 舊 PC 的前 4-bits
* 26-bits jump address
* 00

#### 組合在一起

![](2019-05-03-00-34-27.png)

### Control Unit

#### ALU Control

| ALU control | Function         |
| ----------- | ---------------- |
| 0000        | AND              |
| 0001        | OR               |
| 0010        | add              |
| 0110        | subtract         |
| 0111        | set-on-less-than |
| 1100        | NOR              |

分析一下 Instruction 需要 ALU 做什麼事：

* 分 4 類
  * Load/Store: add
  * Branch: substract
  * R-type: depend on funct-field
* 讀到 10 要再讀 funct field (由 [second level](#second-level-control) 產生出 ALU signal)

| opcode | ALUOp | Operation        | funct  | ALU function     | ALU control |
| ------ | ----- | ---------------- | ------ | ---------------- | ----------- |
| lw     | 00    | load word        | XXXXXX | add              | 0010        |
| sw     | 00    | store word       | XXXXXX | add              | 0010        |
| beq    | 01    | branch equal     | XXXXXX | subtract         | 0110        |
| R-type | 10    | add              | 100000 | add              | 0010        |
|        |       | subtract         | 100010 | subtract         | 0110        |
|        |       | AND              | 100100 | AND              | 0000        |
|        |       | OR               | 100101 | OR               | 0001        |
|        |       | set-on-less-than | 101010 | set-on-less-than | 0111        |

* 25:21: always read
* 20:16: 除了 load 都為 read
* R-type 的 rd, Load 的 rt: write
* address: sign-extended, add

| Instruction | 31:26 | 25:21 | 20:16 | 15:11 | 10:6    | 5:0   |
| ----------- | ----- | ----- | ----- | ----- | ------- | ----- |
| R-type      | 0     | rs    | rt    | rd    | shamt   | funct |
| Load/Store  | 35/43 | rs    | rt    |       | address |       |
| Branch      | 4     | rs    | rt    |       | address |       |

#### R-type

![](2019-05-03-11-08-03.png)

#### Load

![](2019-05-03-11-08-49.png)

#### Branch-on-Equal

![](2019-05-03-11-09-40.png)

#### Jump

![](2019-05-03-11-24-55.png)

#### Control Unit Settings

| Instruction | RegDst | ALUSrc | MemToReg | RegWrite | MemRead | MemWrite | Branch | ALUOp1 | ALUOp0 |
| ----------- | :----: | :----: | :------: | :------: | :-----: | :------: | :----: | :----: | :----: |
| R-format    |   1    |   0    |    0     |    1     |    0    |    0     |   0    |   1    |   0    |
| lw          |   0    |   1    |    1     |    1     |    1    |    0     |   0    |   0    |   0    |
| sw          |   x    |   1    |    x     |    0     |    0    |    1     |   0    |   0    |   0    |
| beq         |   x    |   0    |    x     |    0     |    0    |    0     |   1    |   0    |   1    |

#### First-Level Control

把 Input 的 op 和 Output 的 Control Signal 找出關係：

| Input or output | Signal name | R-format | lw  | sw  | beq |
| --------------- | ----------- | -------- | --- | --- | --- |
| Inputs          | Op5         | 0        | 1   | 1   | 0   |
|                 | Op4         | 0        | 0   | 0   | 0   |
|                 | Op3         | 0        | 0   | 1   | 0   |
|                 | Op2         | 0        | 0   | 0   | 1   |
|                 | Op1         | 0        | 1   | 1   | 0   |
|                 | Op0         | 0        | 1   | 1   | 0   |
| Outputs         | RegDst      | 1        | 0   | X   | X   |
|                 | ALUSrc      | 0        | 1   | 1   | 0   |
|                 | MemtoReg    | 0        | 1   | X   | X   |
|                 | RegWrite    | 1        | 1   | 0   | 0   |
|                 | MemRead     | 0        | 1   | 0   | 0   |
|                 | MemWrite    | 0        | 0   | 1   | 0   |
|                 | Branch      | 0        | 0   | 0   | 1   |
|                 | ALUOp1      | 1        | 0   | 0   | 0   |
|                 | ALUOp2      | 0        | 0   | 0   | 1   |

![](2019-05-03-12-16-50.png)

#### Second-Level Control

將 [ALU Control](#alu-control) 的表格：

| opcode | ALUOp | Operation        | funct  | ALU function     | ALU control |
| ------ | ----- | ---------------- | ------ | ---------------- | ----------- |
| lw     | 00    | load word        | XXXXXX | add              | 0010        |
| sw     | 00    | store word       | XXXXXX | add              | 0010        |
| beq    | 01    | branch equal     | XXXXXX | subtract         | 0110        |
| R-type | 10    | add              | 100000 | add              | 0010        |
|        |       | subtract         | 100010 | subtract         | 0110        |
|        |       | AND              | 100100 | AND              | 0000        |
|        |       | OR               | 100101 | OR               | 0001        |
|        |       | set-on-less-than | 101010 | set-on-less-than | 0111        |

整理一下：

| ALUOp1 | ALUOp0 |  F5   |  F4   |  F3   |  F2   |  F1   |  F0   | ALU control | Operation |
| :----: | :----: | :---: | :---: | :---: | :---: | :---: | :---: | :---------: | :-------: |
|   0    |   0    |   X   |   X   |   X   |   X   |   X   |   X   |    0010     |   lw/sw   |
|   X    |   1    |   X   |   X   |   X   |   X   |   X   |   X   |    0110     |    beq    |
|   1    |   X    |   X   |   X   |   0   |   0   |   0   |   0   |    0010     |    add    |
|   1    |   X    |   X   |   X   |   0   |   0   |   1   |   0   |    0110     |    sub    |
|   1    |   X    |   X   |   X   |   0   |   1   |   0   |   0   |    0000     |    AND    |
|   1    |   X    |   X   |   X   |   0   |   1   |   0   |   1   |    0001     |    OR     |
|   1    |   X    |   X   |   X   |   1   |   0   |   1   |   0   |    0111     |    slt    |

分析 ALU Control 各 bit 的算式，可以發現有些地方結合後其實就是 Don't care：

| ALUOp1 | ALUOp0 |  F5   |  F4   |       F3       |  F2   |  F1   |  F0   |    ALU control    |
| :----: | :----: | :---: | :---: | :------------: | :---: | :---: | :---: | :---------------: |
|   X    |   1    |   X   |   X   |       X        |   X   |   X   |   X   | 0<mark>1</mark>10 |
|   1    |   X    |   X   |   X   | <mark>0</mark> |   0   |   1   |   0   | 0<mark>1</mark>10 |
|   1    |   X    |   X   |   X   | <mark>1</mark> |   0   |   1   |   0   | 0<mark>1</mark>11 |

$ALUctr2 = ALUop0 + ALUop1 \cdot func2' \cdot func1 \cdot func0'$

| ALUOp1 |     ALUOp0     |  F5   |  F4   |       F3       |  F2   |       F1       |  F0   |    ALU control    |
| :----: | :------------: | :---: | :---: | :------------: | :---: | :------------: | :---: | :---------------: |
|   0    | <mark>0</mark> |   X   |   X   |       X        |   X   |       X        |   X   | 00<mark>1</mark>0 |
|   X    | <mark>1</mark> |   X   |   X   |       X        |   X   |       X        |   X   | 01<mark>1</mark>0 |
|   1    |       X        |   X   |   X   | <mark>0</mark> |   0   | <mark>0</mark> |   0   | 00<mark>1</mark>0 |
|   1    |       X        |   X   |   X   | <mark>0</mark> |   0   | <mark>1</mark> |   0   | 01<mark>1</mark>0 |
|   1    |       X        |   X   |   X   | <mark>1</mark> |   0   | <mark>1</mark> |   0   | 01<mark>1</mark>1 |

$ALUctr1 = ALUop1' + ALUop1 \cdot func2' \cdot func0'$

| ALUOp1 | ALUOp0 |  F5   |  F4   |  F3   |  F2   |  F1   |  F0   |    ALU control    |
| :----: | :----: | :---: | :---: | :---: | :---: | :---: | :---: | :---------------: |
|   1    |   X    |   X   |   X   |   0   |   1   |   0   |   1   | 000<mark>1</mark> |
|   1    |   X    |   X   |   X   |   1   |   0   |   1   |   0   | 011<mark>1</mark> |

$ALUctr0 = ALUop1 \cdot func3' \cdot func2 \cdot func1' \cdot func0 + ALUop1 \cdot func3 \cdot func2' \cdot func1 \cdot func0'$

![](2019-05-03-12-36-54.png)

### Performance

* Critical path: load instruction
  * **Instruction memory $\to$ register file $\to$ ALU $\to$ data memory $\to$ register file**
* Violates design principle: Making the common case fast

### Pipeline

Parallelism improves performance.

![](2019-05-08-19-23-57.png)

#### MIPS Pipeline

5個 stage 同時執行不同指令

* IF: Instruction fetch from memory
* ID: Instruction decode & register read
* EX: Execute operation or calculate address
* MEM: Access memory operand
* WB: Write result back to register

最理想: 一個 cycle 執行完一個指令

![Single-cycle (Tc = 800ps) vs Pipelined (Tc = 200ps)](2019-05-09-01-04-36.png)

MIPS ISA designed for pipelining:

* 所有 instruction 都為 32-bits
  * 較容易在一個 cycle 中做 fetch 及 decode
  * c.f. x86: 1- to 17-byte instructions
* Instruction formats 少
  * 較容易在一個 step 中做 decode 及 read registers
* Load/store addressing
  * calculate address in $3^{rd}$ stage
  * access memory in $4^{th}$ stage
* Alignment of memory operands
  * Memory access 只需要 1 個 cycle

#### Performance

若所有 stage 都花費同樣時間，則

$$ \text{Time between instructions}_{\text{pipelined}} = \frac{ \text{Time between instructions}_{\text{nonpipelined}} }{ \text{Number of stage} } $$

能 speed up 的原因是因為 [throughput](#response-time-and-throughput) 上升，latency (time for each instruction) 不變

#### Hazards

1. <mark>Structure hazards</mark>: 硬體架構，大家都要用但只有一份resouse，要用就要排隊
2. <mark>Data hazard</mark>: 前一個指令算完時還讀不到 (前一個指令第5個stage才存，但我第2個stage就要)
3. <mark>Control hazard</mark>: 下一個clock cycle不知道要執行甚麼指令

#### Summary

* pipeline 增加 throughput
  * 一個時間點每個 stage 執行不同指令
  * 每個 instruction 有相同的 latency
* performance 被 hazard 限制
* ISA 影響 pipeline 實做的複雜度

### Pipeline Datapath

![由右往左的 dataflow 常有 hazard 問題](2019-05-08-23-53-11.png)

同一條線要怎麼 keep 3 個值？加上 <mark>pipeline register</mark>！以保持指令資訊

![](2019-05-08-23-54-38.png)

### Pipeline Register

![](2019-05-09-00-03-48.png)

![](2019-05-09-00-03-59.png)

![](2019-05-09-00-04-13.png)

![](2019-05-09-00-04-26.png)

![](2019-05-09-00-05-52.png)

寫入的是第 $i + 3$ 個 write register，不是第 $i$ 個的 write register！

![](2019-05-09-00-07-09.png)

**Solution**: <mark>把 write register pack 起來</mark>，帶著一起走

### Pipeline Operation

* single-clock-cycle pipeline diagram: 分析一個 cycle 的 pipeline usage
* muli-cycle pipline diagram: 分析各 operation 在不同 cycle 的 resource usage

#### Single-Cycle diagram

pipeline 在某個 cycle 的運作情形

![](2019-05-09-00-10-41.png)

#### Multi-Cycle Diagram

showing resource usage

![](2019-05-09-00-09-25.png)

### Pipeline Control

![](2019-05-09-00-20-43.png)

> Q: 為甚麼不把 RegDst 搬到前面，可以少用 5-bits?
> A: MUX 需要 input selection (RegDst) ，由 control unit 產生，如果放在第二個 stage RegDst 還沒產生

Control values 基本上與 [Single Cycle CPU - ALU Control](#alu-control) 沒有什麼區別

| opcode | ALUOp | Operation        | funct  | ALU function     | ALU control |
| ------ | ----- | ---------------- | ------ | ---------------- | ----------- |
| lw     | 00    | load word        | XXXXXX | add              | 0010        |
| sw     | 00    | store word       | XXXXXX | add              | 0010        |
| beq    | 01    | branch equal     | XXXXXX | subtract         | 0110        |
| R-type | 10    | add              | 100000 | add              | 0010        |
|        |       | subtract         | 100010 | subtract         | 0110        |
|        |       | AND              | 100100 | AND              | 0000        |
|        |       | OR               | 100101 | OR               | 0001        |
|        |       | set-on-less-than | 101010 | set-on-less-than | 0111        |

Recall: [Single Cycle CPU - Control Unit Settings](#control-unit-settings)，將不同 stage 的 control signal 整理一下：

| Stage    | Instruction | R-format | lw  | sw  | beq |
| -------- | ----------- | -------- | --- | --- | --- |
| EX stage | RegDst      | 1        | 0   | x   | x   |
|          | ALUOp1      | 1        | 0   | 0   | 0   |
|          | ALUOp0      | 0        | 0   | 0   | 1   |
|          | ALUSrc      | 0        | 1   | 1   | 0   |
| M stage  | Branch      | 0        | 0   | 0   | 1   |
|          | MemRead     | 0        | 1   | 0   | 0   |
|          | MemWrite    | 0        | 0   | 1   | 0   |
| WB stage | RegWrite    | 1        | 1   | 0   | 0   |
|          | MemToReg    | 0        | 1   | x   | x   |

用到的就可以不要了，沒用到的繼續傳下去

![](2019-05-09-00-35-32.png)

RegWrt 要滿足打包的原則，所以會被 pack 到 pipeline regiter

![](2019-05-09-00-40-36.png)


### Pipeline Hazards

#### Data Hazards

* **Solution.1** <mark>bubble(no opertation)</mark>
  * panelty: 50%的使用率 (第一個指令做完 > no operation > no operation > 第二個指令執行完畢)

![](2019-05-08-23-11-26.png)

* **Solution.2**: <mark>forwarding(bypassing)</mark>
  * 只要拿到值就開始算，不用等存到 reg 再讀出來

![](2019-05-08-23-12-35.png)

##### Example

![](2019-05-09-00-48-16.png)

sub 在 cycle 5 才將資料寫回 register，用紅線的 forwarding 解決

#### Load-Use Data Hazard

![](2019-05-08-23-17-49.png)

* Load & R-type
* lw 在第四個 stage 結束後才將 data 讀出來
* EX 後是 memory address 不是 data，不能直接拉 EX 後那條

**Solution**: <mark>reorder code</mark>，讓 lw 與 add 間隔兩個 cycle

##### Example

![](2019-05-08-23-20-54.png)

#### Control Hazards

beq 下一個 Instruction fetch 不確定要執行甚麼指令，不知道該跳還是不要跳 (depend on branch 的結果)

![](2019-05-08-23-22-43.png)

**Solution**: <mark>猜!</mark>

![](2019-05-08-23-24-23.png)

猜不要跳，猜對了

![](2019-05-08-23-34-02.png)

猜不要跳，lw 正要 pipeline instruction fetch (但不應該被執行)，然後發現猜錯了，於是讓 lw 後面都變成 no operation (這邊加上一些硬體的設定直接 flush 掉)，接著跳到 or 繼續執行

有兩種猜的方法：

* Static branch prediction: 每次都猜一樣的
* Dynamic branch prediction: 上次猜對還猜錯，猜對了繼續猜同樣的，猜錯了跟個性有關，有猜錯一次就換的，也猜錯多次才換的

### p83

偵測 load/use hazard

### p84

第2個指令被decode兩次
第3個指令被fetch兩次

### p85

and become nop: stall 一個 cycle
or 不需要 forward，直接讀

### p86

進到 cycle 3: detect load/sw hazards

IF/ID 紀錄第二個指令，PC只到第三個指令

第三個指令(or)再cc3讀一次cc4又讀一次
第二個指令再cc3 decode依次cc4又 decode 依次

### p87

hazard detect unit

control 在發現 hazard 時全部灌 0
PCWrite = 0，不讓 PC + 4 寫進 PC
IF/IDWrite = 0，不讓 instruction 寫入

### p89

正確的指令在第5個 cycle 進來，前面全部 flush 掉(不keep)

### p90

提早發現 branch 跳錯，搬到第2個stage

* 加 target address adder
* 加 register comparision

第2個stage結束後就可以發現 hazard

### p93

又有 hazard 又有 branch

### p96

上次猜甚麼 猜對還猜錯

### p97

depend on iteration numbers

