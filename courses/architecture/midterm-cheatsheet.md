# Cheetsheat for Architecture Midterm

### Lec.01: basic concepts

Von Neumann Architecture: separate processor/memory

Code Compilation & Translation:
High-level(\*.c) -> Assembly(\*.s) -> Machine object(\*.o) -> executable

Processors: realizes ISA (RISC vs CISC)

Registers

Memory: Byte-Oriented Memory Accessing

- Big-endian & little-endian (significant byte -> lower/higher addr.)

Memory mapping: Stack(calling, the bottom has high address), Heap(dynamic data)



### Lec.02: Metrics

Performance: latency(time for one task) & throughput(# tasks per unit time)

- Performance = (Execution time)$^{-1}$
- Speedup of X over Y = Perf$_X$ / Perf$_Y$.
- Execution time = #Cycles $\times$ CCT(clock cycle time)

Normally, we can use #Cycles to approximate exe. time.

CPI: cycles per instruction

- #Cycles = IC $\times$ CPI, IC = instruction count in one program
- Execution time = IC $\times$ CPI $\times$ CCT

- Average CPI = $\sum$ IC / IC$_i$ $\times$ CPI$_i$. 

Impact of memory, i.e., data accesses, could be reflected on CPI. Roofline model:

Means of metrics: Amean for absolutes; Hmean for rates; Gmean for ratios

Power & energy efficiency

Scaling:

- Strong scaling: speedup on N processors with fixed total workload size
- Weak scaling: speedup on N processors with fixed per-processor workload size

- Limitation: imbalanced load; Amdhal's Law: an optimization accelerates a fraction $f$ of a program by a factor of $S$, then speedup = $1/(1-f+f/S)$.

### Lec.03 RISC-V ISA and Assembly

RISC-V system states: PC, 32 registers `x0`~`x31`(alias), memory from 0 to MSIZE-1 
Special registers: `ra`, `tp`, `sp`, `gp`.

Some details:

- 12-bit signed offset in `lw` and `sw` instruction; (most case)
- 12-bit immediate in most instructions; 
- 20-bit immediate in `lui` and `auipc` for loading the whole immediate.



### Lec.04 RISC-V encoding

Six instruction formats:

- R-format: register-register operations 
- I-format: register-immediate operations, loads, jalr (12-bit imm, 11:0)
- S-format: stores (12-bit imm, 11:0) 
- B-format: branches (12-bit imm, 12:1)
- J-format: jumps (20-bit imm, 20:1) 
- U-format: upper immediate instructions (20-bit imm, 31:12)



ISA for parallelism

- SIMD: data-level parallelism
  - Example: Intel x86 SSE and AVX Extensions
  - SIMD intrinsics: use C functions instead of inline assembly to call SSE/AVX instructions

- MIMD: thread-level parallelism
  - process and thread
  - Shared-memory multi-core processors
  - ISA: Atomic instructions (A extension)
  - Another possibility: using locks



### Lec.05 Decoding & processor

Simple format of RISC-V $\to$ simple instruction decoding

Multi-level decoding:

<img src="https://wu-ys.github.io/courses/architecture/midterm-cheatsheet.assets/image-20220418195328525.png" alt="image-20220418195328525" style="zoom: 30%;" />



Datapath & control bits

Structure & critical path:

<img src="https://wu-ys.github.io/courses/architecture/midterm-cheatsheet.assets/image-20220418195554733.png" alt="image-20220418195554733" style="zoom: 80%;" />

Pipelining: 5 stages IF/ID/EX/MEM/WB

<img src="https://wu-ys.github.io/courses/architecture/midterm-cheatsheet.assets/image-20220418200214694.png" alt="image-20220418200214694" style="zoom:50%;" />

### Lec.06 Avoiding hazards and stalls

Stalls: a pipeline “bubble”, not moving forward in that cycle 

- How to stall: insert `nop` instructions, or let the hardware stall as needed.

- Causes of stalls: hazards

Hazards:

- Structural hazard: A required resource is busy and cannot be used in this cycle 
  - different read/write ports; half-cycle register access
  - cancel stage bypassing
  - seperated memory(IM, DM)
  - let multi-stage instructions go
- Data hazard: Must wait previous dependent instructions to produce/consume data (RAW in typical)
  - Solution: forwarding
  - EX, MEM, WB $\to$ EX, MEM
  - forwarding control logic
  - exchange the order of instructions to avoid stalls
- Control hazard: Next PC depends on previous instruction
  - Resolving branch/jump earlier (in ID stage)
  - Furthermore: predict the next PC (usually, PC+4)
  - In case of misprediction, we need to nullify the operations and restart as soon as possible.
  - Additional forwarding to ID for branches



## Lec.07 Advanced Techniques

### Branch prediction

Key idea of hardware-based prediction: predict based on recent history

Predicting direction: 

- A hardware branch history table (BHT) records each branch’s history

- Each branch maintains a saturate counter to predict the next direction 
  - Counter +1 for an actually taken branch, -1 for not-taken
  - 1-bit states: 0: predict not-taken, 1: predict taken 
  - 2-bit states: 00: predict not-taken, 01: not-taken?, 10: taken?, 11: taken
  - Simple 2-bit BHT already achieves > 90% accuracy for most programs

Predicting target: Branch target buffer (BTB)

- Record previously executed branches and their most recent target addresses 
- Such a hardware table is implemented using SRAM 
  - Address is branch instruction address, i.e., current PC 
  - Data is the most recent target, i.e., next PC

Overall workflow

- At IF stage, if BHT predicts to be not taken, then PC = PC + 4 
- Else, use current PC to index into BTB 
- If there is an entry, then PC = target address in BTB, else PC = PC + 4

![image-20220418204125010](D:\wys\github-website\wu-ys20\courses\architecture\midterm-cheatsheet.assets\image-20220418204125010-16503653810021.png)

### Superscalar and Out-of-Order

ILP: Instruction-Level Parallelism = critical path / # instructions

Superscalar execution: multi-instructions per cycle 

Out-of-Order (OoO) Execution

- Use diversified functional units to finish instructions ASAP to avoid blocking
- Use inter-stage buffers for decoupling stages and reordering instructions

Stages: Fetch

- decode: also read registers
- Dispatch/rename/allocation: Resolve dependencies and prepare for OoO execution 

- Issue/schedule: Determine whether instruction is ready to execute 
- Execute: actual execution of instructions, e.g., ALU, memory access
- Commit/retire/write-back: Update architectural states (registers or memory)
- Each stage can me one or more cycles