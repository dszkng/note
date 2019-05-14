# Computer Abstractions and Technology

## Abstractions architecture

![](2019-04-17-21-22-13.png)

ISA: Instruction set architecture (The hardware/software interface)

## Trade-off

### Power Trends

$$
Power = Capacitive \, load \times Voltage^2 \times Frequency
$$

### Integrated Circuit Cost

$$
Cost \, per \, die = \frac{ Cost \, per \, wafer }{ Dies \, per \, wafer * Yield}
$$

$$
Dies \, per \, wafer \approx \frac{Wafer \, area}{Die \, area}
$$

$$
Yield = \frac{1}{(1 + (Defects \, per \, area * Die \, area / 2))^2}
$$

## Performance

### Relative Performance

$$
Performance = \frac{1}{Execution \, Time}
$$

e.g. X is n time faster than Y

$$
\frac{Performance_X}{Performance_Y} = \frac{Execution \, time_Y}{Execution \, time_X} = n
$$

### Response Time and Throughput

* **Response time**(Elapsed time): The time it takes to do a task（**要完成一件工作，所需花費的時間**）
    * including all aspects (Processing, I/O, OS overhead, idle time)
    * Determines system performance
* **Throughput**: Total work done per unit time（**在一定時間內，所能完成的工作量**）
    * e.g., tasks/transactions/… per hour

### CPU Clocking

![](2019-04-17-21-33-51.png)

* Clock period: duration of a clock cycle（一個 Cycle 花費的時間）
    * e.g., 250ps = 0.25ns = 250×10 –12 s
* Clock frequency (rate): cycles per second（一秒幾個 Cycle）
    * e.g., 4.0GHz = 4000MHz = 4.0×10 9 Hz

### CPU time

**CPU處理給定的工作所花費的時間** (Discounts I/O time, other jobs' shares)

$$
CPU \, Time = Clock \, Cycles \times Clock \, Cycle \, Time = \frac{CPU \, Clock \, Cycles}{Clock \, Rate}
$$

公式理解：**處理工作時走了幾個 Cycle** 乘上**每走一個 Cycle 所要花費的時間**

* Performance improved by
    * Reducing number of clock cycles
    * Increasing clock rate
    * Hardware designer must often trade off clock rate against cycle count

### Cycles per Instruction (CPI)

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

### Performance Summary

$$
CPI = \frac{Instructions}{Program} \times \frac{Clock \, cycles}{Instruction} \times \frac{Seconds}{Clock \, cycle}
$$

* Performance depends on
    * Algorithm: **affects IC**, possibly CPI
    * Programming language: **affects IC, CPI**
    * Compiler: **affects IC, CPI**
    * Instruction set architecture: **affects IC, CPI, Tc**

## [Amdahl's Law](https://en.wikipedia.org/wiki/Amdahl%27s_law)

Improving an aspect of a computer and expecting a proportional improvement in overall performance

$$
T_{improved} = \frac{T_{affected}}{Improvement \, factor} + T_{unaffected}
$$

## MIPS

Millions of Instructions Per Second

$$
MIPS = \frac{Instruction \, count}{Execution \, time \times 10^6} = \frac{Instruction \, count}{\frac{Instruction \, count \times CPI}{Clock \, rate} \times 10^6} = \frac{Clock rate}{CPI \times 10^6}
$$

> ps: MIPS無法拿來當作電腦效能的依據（**因為沒有考慮到 Instruction Count**）