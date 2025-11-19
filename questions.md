# ARM Processor Exam Question Bank & Study Guide

## 1. Emulation Logic (Instruction Set Simulation)

### Questions
1.  **Emulation Loop Structure:** Write the pseudo-code structure for an ARM-based emulator that interprets Intel x86 instructions. It must show the Fetch, Decode, and Execute stages.
2.  **Register Mapping:** Explain how you would map the Intel 8086 register set (`AX`, `BX`, `CX`, `DX`, `SI`, `DI`, `SP`, `BP`, `IP`, `Flags`) to the ARM register set (`R0`-`R15`) for an efficient emulator. Which ARM registers should be reserved for the emulator's own use?
3.  **Pipeline Simulation:** Real ARM processors have a pipeline (Fetch, Decode, Execute). Your simple emulator loop is sequential. Modify the standard `WHILE` loop pseudo-code to simulate a 3-stage pipeline. Use three variables: `Instruction_Fetch`, `Instruction_Decode`, `Instruction_Execute`.
4.  **Condition Codes:** Intel and ARM both use condition flags (N, Z, C, V). Write a macro `UPDATE_FLAGS(Result, Op1, Op2)` that calculates the **Overflow (V)** flag for a 32-bit signed addition emulation.

### Where to Study
* **Concept:** Lecture notes on "Instruction Set Simulation" (ISS).
* **Logic:** `ARM_Exam_Notes.md` (Section 1.1).
* **Reference:** General computer architecture textbooks (e.g., Patterson & Hennessy) on "Interpretation".

---

## 2. Instruction Translation (Coding)

### Questions
1.  **Intel `REP MOVS` Translation:** Write the ARM assembly code to emulate the Intel `REP MOVS` instruction. (Copy a byte from `[SI]` to `[DI]`, increment both, decrement `CX`, repeat until `CX` is 0).
2.  **Intel `POP` Emulation:** Write an ARM assembly macro `EMU_INTEL_POP(R_Dest)` to emulate the Intel x86 `POP` instruction. (Hint: Intel stack grows down, so POP reads then increments SP).
3.  **Intel `XCHG` Translation:** The Intel `XCHG EAX, EBX` instruction swaps the contents of two registers atomically. Write an ARM assembly sequence to emulate this using `R0` (EAX) and `R1` (EBX). Can you do this without a temporary register?
4.  **Intel `LOOP` Translation:** The Intel `LOOP Label` instruction decrements `CX` and branches to `Label` if `CX` is not zero. Write the ARM assembly equivalent using `R2` as `CX`.
5.  **TMS320 `MAC` Emulation:** Write an ARM macro `EMU_MAC(PtrA, PtrB, Accumulator)` to emulate the TMS320 `MAC *AR0+, *AR1+, A` instruction. (Multiply values at pointers, add to Acc, post-increment pointers).
6.  **TMS320 `LDI` Emulation:** You are writing an emulator for a TMS320 DSP. The DSP has a specialized instruction `LDI *AR0++, R1` (Load Indirect with Post-Increment). Which single ARM instruction performs this exact operation?
7.  **TMS320 Circular Buffer:** A TMS320 DSP uses circular addressing for a buffer of size 256 bytes. The pointer `AR0` wraps around automatically. Write ARM pseudo-code to emulate: `LDR R0, [AR0++]` with circular wrapping logic.
8.  **Intel `ADD [Mem], Reg`:** Write the ARM instruction sequence to emulate the Intel CISC instruction `ADD [EBX], EAX`. (Hint: Load -> Add -> Store).

### Where to Study
* **ARM Instructions:** `ARMv7-AR_TRM.pdf` (Part A, Chapter A8). Specifically `LDR`, `STR`, `ADD`, `SUB`, `CMP`, `B`.
* **Addressing Modes:** `ARMv7-AR_TRM.pdf` (Part A, Chapter A5). Read about Pre-indexed vs. Post-indexed addressing.
* **Logic:** `ARM_Exam_Notes.md` (Section 1.2 & Practice Questions).

---

## 3. Hardware Mechanics & Exceptions (System)

### Questions
1.  **IRQ Entry Sequence:** Write the exact 5-step hardware pseudo-code sequence that occurs when an **IRQ (Interrupt Request)** is triggered. Use register assignment syntax (e.g., `Reg = Value`).
2.  **Data Abort Entry Sequence:** Write the 5-step hardware pseudo-code for a **Data Abort** exception. (Hint: Abort mode is `10111`, Vector is `0x10`, Return is `PC+8`).
3.  **FIQ vs. IRQ:** Write the 5-step hardware pseudo-code sequence for **FIQ Entry**. Explain *why* FIQ is faster than IRQ. Specifically mention the banked registers `R8_fiq` through `R14_fiq`.
4.  **Reset Exception:** A **Reset** exception occurs. What is the binary mode encoding for Supervisor Mode (SVC)? Write the pseudo-code for the PC update. Where does execution start?
5.  **Undefined Instruction:** The processor attempts to decode an instruction bit pattern that does not match any known opcode. Write the hardware sequence to enter the **Undefined** exception. Which register holds the address of the 'bad' instruction?
6.  **Register Banking:** Explain why ARM switches to a different Stack Pointer (`R13_irq`) when an IRQ occurs. What would happen if it shared the User Mode stack pointer?
7.  **SPSR Function:** Explain the function of the **SPSR** register. Why is it not available in User Mode?

### Where to Study
* **Exception Logic:** `ARMv7-AR_TRM.pdf` (Part B, Chapter B1, Section "Exception Entry").
* **Mode Bits:** `ARMv7-AR_TRM.pdf` (Part B, Chapter B1, Section "ARM Processor Modes").
* **Summary:** `ARM_Exam_Notes.md` (Section 2).

---

## 4. Intel & TMS Specifics (Context for Emulation)

### Questions
1.  **Intel Bus Bandwidth:** An Intel 80C186 is operating at 16 MHz. It needs to transfer a 64KB block of data with 2 wait states per transfer. Calculate the total time required for a 16-bit bus vs. an 8-bit bus.
2.  **Intel Serial Comm:** Two 80C186 processors communicate via serial link (9600 bps). Calculate the minimum time to transfer 512 bytes.
3.  **Intel Interrupt Latency:** Calculate the worst-case interrupt latency for an 80C186 executing a 40-cycle `DIV` instruction.
4.  **TMS FIR Filter:** Write a TMS320C31 subroutine to implement a 4-tap FIR filter using parallel `MPYF3 || ADDF3` instructions.
5.  **TMS Matrix Mult:** Develop an assembler subroutine to multiply a 3x3 matrix by a 3x1 vector.
6.  **Stack vs. Heap:** A DSP application calls a recursive function 500 times (4 words/frame). Calculate the minimum stack size required.

### Where to Study
* **Intel Specs:** `EE316 Minor2 sols.pdf` & `EE316 Major sols (1).pdf` (Check the solutions for timing calculations).
* **TMS Specs:** `TMS320C3x User's Guide` (Chapters 3 & 8).

```
```
