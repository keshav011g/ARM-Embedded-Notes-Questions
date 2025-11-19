# ARM Processor Exam Notes: Emulation, Translation & Architecture

## Table of Contents

- [1. Instruction Set Simulation (ISS) & Translation](#1-instruction-set-simulation-iss--translation)
    - [1.1 The Emulation Loop](#11-the-emulation-loop-the-fetch-decode-execute-cycle)
    - [1.2 Translation Challenges](#12-translation-challenges-cisc-vs-risc-vs-dsp)
- [2. ARM Architecture Hardware Mechanics](#2-arm-architecture-hardware-mechanics)
    - [2.1 Processor Modes](#21-processor-modes)
    - [2.2 Exception Entry Sequence](#22-exception-entry-sequence-critical)
    - [2.3 Registers](#23-registers)
- [3. Practice Test Questions](#3-practice-test-questions)
    - [Section A: Emulation Coding](#section-a-emulation-coding)
    - [Section B: Hardware Logic](#section-b-hardware-logic)
- [4. Solutions](#4-solutions)

---

## 1. Instruction Set Simulation (ISS) & Translation

This is the primary theme of your course: using a **Host** processor (ARM) to execute code designed for a **Guest** processor (Intel x86 or TMS320 DSP).

### 1.1 The Emulation Loop (The "Fetch-Decode-Execute" Cycle)

You must be able to write this pseudo-code structure from memory. It simulates the hardware's instruction cycle in software.

[Image of fetch-decode-execute cycle diagram]

```c
// Define Virtual Registers for the Guest CPU
INT32 vEAX, vEBX, vECX, vEDX;  // Intel Registers
INT32 vPC;                     // Virtual Program Counter
INT32 vFlags;                  // Virtual Status Register

// The Emulator Loop
WHILE (Emulator_Is_Running) DO:
    
    // 1. FETCH
    // Read the instruction opcode from the guest memory image
    Opcode = Guest_Memory[vPC];
    
    // 2. DECODE
    // Determine which instruction it is (Switch Statement)
    SWITCH (Opcode):
    
        CASE INTEL_MOV_EAX_EBX:
            // 3. EXECUTE
            // Perform the operation on virtual registers
            vEAX = vEBX;
            vPC = vPC + 1; // Increment Virtual PC
            BREAK;
            
        CASE INTEL_ADD_MEM_EAX:
            // Complex CISC instruction: ADD [Address], EAX
            // Requires multiple host steps: Load -> Add -> Store
            Temp_Addr = Fetch_Address_Operand();
            Temp_Data = Guest_Memory[Temp_Addr];
            Temp_Data = Temp_Data + vEAX;
            Guest_Memory[Temp_Addr] = Temp_Data;
            Update_Virtual_Flags(Temp_Data);
            vPC = vPC + Instruction_Length;
            BREAK;
            
        DEFAULT:
            Handle_Undefined_Opcode();
            BREAK;
            
    END SWITCH
END WHILE
```

### 1.2 Translation Challenges: CISC vs. RISC vs. DSP

* **Intel x86 (CISC) on ARM (RISC):**
    * **Problem:** Intel allows arithmetic directly on memory (e.g., `ADD [EBX], EAX`). ARM is Load-Store (arithmetic only on registers).
    * **Solution:** You must break one Intel instruction into a sequence: `LDR` (load from memory) $\rightarrow$ `ADD` (do math) $\rightarrow$ `STR` (store back).
    * **Stack:** Intel `PUSH` decrements SP *then* stores. ARM `STMDB` (Store Multiple Decrement Before) matches this behavior perfectly.

* **TMS320 (DSP) on ARM:**
    * **Problem:** DSPs have specialized hardware for math, like `MAC` (Multiply-Accumulate) and circular buffers.
    * **Solution:** Use ARM's `MLA` (Multiply Accumulate) instruction.
    * **Address Generation:** TMS320 uses post-increment (e.g., `*AR0++`). ARM supports this natively: `LDR R0, [R1], #4`.

---

## 2. ARM Architecture Hardware Mechanics

This section covers the specific hardware details from your `ARMv7-A/R` manual that appeared in your exam questions.

### 2.1 Processor Modes

ARM has several modes of operation. You need to know the binary codes for the **Program Status Register (CPSR)**.

| Mode | Description | Binary Encoding (M[4:0]) |
| :--- | :--- | :--- |
| **User** | Normal program execution | `10000` |
| **FIQ** | Fast Interrupt | `10001` |
| **IRQ** | Standard Interrupt | `10010` |
| **Supervisor (SVC)** | Reset or Soft Interrupt (SWI) | `10011` |
| **Abort** | Memory access violation | `10111` |
| **Undefined** | Unknown instruction | `11011` |
| **System** | Privileged mode (shares User regs) | `11111` |

### 2.2 Exception Entry Sequence (CRITICAL)

This is the exact hardware logic for "What happens when an interrupt occurs?". **Memorize this.**

**Scenario:** An **IRQ** (Interrupt Request) occurs.

1.  **Save Return Address:** `LR_irq = PC + 4` (Adjusts for pipeline; +4 for IRQ, +8 for Data Abort).
2.  **Save Current Status:** `SPSR_irq = CPSR` (Back up current flags/mode).
3.  **Change Processor Mode:** `CPSR[4:0] = 0b10010` (Switch mode bits to IRQ).
4.  **Disable Interrupts:** `CPSR[7] = 1` (Set the 'I' bit to mask further IRQs).
5.  **Jump to Vector:** `PC = 0x00000018` (Load the IRQ vector address).

### 2.3 Registers

* **R0-R12:** General purpose.
* **R13 (SP):** Stack Pointer (Banked in exception modes).
* **R14 (LR):** Link Register (Holds return address).
* **R15 (PC):** Program Counter.
* **CPSR:** Current Program Status Register (Flags N, Z, C, V + Control Bits).
* **SPSR:** Saved Program Status Register (Only exists in exception modes).

---

## 3. Practice Test Questions

### Section A: Emulation Coding

**Q1:** Write an ARM assembly macro `EMU_INTEL_POP(R_Dest)` to emulate the Intel x86 `POP` instruction.
* *Hint:* Intel `POP` reads from `[SP]` then increments `SP` by 4. Assume `R13` is the virtual SP.

**Q2:** Write the ARM pseudo-code loop to emulate an Intel `REP STOS` instruction.
* *Context:* `REP STOS` stores the value in `EAX` to the memory at `[DI]`, increments `DI`, and decrements `CX` until `CX` is 0.
* *Virtual Regs:* `R0`=EAX, `R1`=DI, `R2`=CX.

### Section B: Hardware Logic

**Q3:** A **Data Abort** exception occurs. Write the 5-step hardware pseudo-code sequence.
* *Specifics:* Abort Mode code is `10111`. Vector address is `0x00000010`. Return adjustment is `PC+8`.

**Q4:** You are writing an emulator for a TMS320 DSP. The DSP has a specialized instruction `LDI *AR0++, R1` (Load Indirect with Post-Increment).
* *Question:* Which single ARM instruction performs this exact operation? Assume `R0` holds the address (AR0) and `R1` is the destination.

**Q5:** Explain the function of the **SPSR** register. Why is it not available in User Mode?

---

## 4. Solutions

**A1: Intel POP Emulation**

```assembly
MACRO EMU_INTEL_POP(R_Dest)
    LDR R_Dest, [R13], #4  ; Load from [SP], then Post-Increment SP by 4
END MACRO
```
*Note: Intel stack grows down (PUSH decrements) and shrinks up (POP increments).*

**A2: REP STOS Emulation**

```text
Loop_Start:
    CMP R2, #0              ; Check if CX is 0
    BEQ Loop_End            ; If 0, done
    
    STR R0, [R1], #4        ; Store EAX to [DI], Post-Increment DI
    SUB R2, R2, #1          ; Decrement CX
    B Loop_Start            ; Repeat
Loop_End:
```

**A3: Data Abort Hardware Logic**

1.  `LR_abt = PC + 8`
2.  `SPSR_abt = CPSR`
3.  `CPSR[4:0] = 0b10111`
4.  `CPSR[7] = 1` (Disable IRQ)
5.  `PC = 0x00000010`

**A4: TMS320 LDI Emulation**

The equivalent ARM instruction is:
`LDR R1, [R0], #4`
*(Load R1 from address in R0, then add 4 to R0)*.

**A5: SPSR Explanation**

* **Function:** The **Saved Program Status Register** stores the `CPSR` (flags and mode) of the *previous* mode when an exception occurs. This allows the CPU to restore the original state (return from interrupt) by copying `SPSR` back to `CPSR`.
* **User Mode:** User mode is not entered via exception, so it has no "previous state" to save. Therefore, it has no banked SPSR.
```
```
