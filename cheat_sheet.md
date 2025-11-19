# ARM Exam Reference Guide (Open Book)

## 1. Hardware Mechanics "Cheat Sheet"

### 1.1 Exception Entry Sequence (Memorize)
When an exception occurs (e.g., IRQ), the hardware performs these 5 steps automatically:

1.  **Save Return Address:** `LR_<mode> = PC + 4` (or `+8` for Data Abort)
2.  **Save Status:** `SPSR_<mode> = CPSR`
3.  **Change Mode:** `CPSR[4:0] = <Mode_Binary>`
4.  **Disable Interrupts:** `CPSR[7] = 1` (Sets 'I' bit)
5.  **Jump to Vector:** `PC = <Vector_Address>`

### 1.2 Processor Modes & Vectors
| Mode | Encoding (M[4:0]) | Vector Address | Note |
| :--- | :--- | :--- | :--- |
| **User** | `10000` | - | Normal execution |
| **FIQ** | `10001` | `0x0000001C` | Fast Interrupt |
| **IRQ** | `10010` | `0x00000018` | Standard Interrupt |
| **SVC** | `10011` | `0x00000008` | Supervisor (SWI/Reset) |
| **Abort** | `10111` | `0x00000010` | Data Abort (Memory Fault) |
| **Undef**| `11011` | `0x00000004` | Undefined Instruction |

[Image of ARM Exception Vectors table]

---

## 2. Emulation Templates (Recipes)

### 2.1 The Emulator Loop (Pseudo-code)
```text
WHILE (Running)
    Opcode = Memory[Virtual_PC]         // 1. Fetch
    Virtual_PC++
    
    SWITCH (Opcode)                     // 2. Decode
        CASE INTEL_MOV: 
            Do_Mov(); BREAK;
        CASE TMS_MAC:   
            Do_Mac(); BREAK;
    END SWITCH                          // 3. Execute
END WHILE
```

### 2.2 Translation: Intel (CISC) to ARM (RISC)
* **Rule:** Intel ops on memory (`ADD [mem], Reg`) must be split: `Load -> Operate -> Store`.
* **Intel `PUSH EAX`**:
    ```assembly
    STR R0, [R13, #-4]!   ; Pre-decrement SP (R13), store R0 (EAX)
    ```
* **Intel `POP EAX`**:
    ```assembly
    LDR R0, [R13], #4     ; Load R0 (EAX), Post-increment SP (R13)
    ```
* **Intel `REP MOVS` (String Copy)**:
    ```text
    Loop:
        CMP R3, #0          ; Check CX
        BEQ Done
        LDRB R0, [R1], #1   ; Load [SI] -> R0, SI++
        STRB R0, [R2], #1   ; Store R0 -> [DI], DI++
        SUB R3, R3, #1      ; CX--
        B Loop
    Done:
    ```

### 2.3 Translation: TMS320 (DSP) to ARM
* **TMS `MAC` (Multiply-Accumulate)**:
    ```assembly
    LDR R_Temp1, [R_PtrA], #4   ; Load & Post-Inc
    LDR R_Temp2, [R_PtrB], #4   ; Load & Post-Inc
    MLA R_Acc, R_Temp1, R_Temp2, R_Acc ; Mult-Accumulate
    ```
* **TMS Circular Buffer (Pseudo-code)**:
    ```text
    LDR R0, [R_Ptr]             // Load data
    R_Ptr = R_Ptr + 4           // Increment pointer
    IF R_Ptr >= Buffer_End THEN // Check wrap-around
        R_Ptr = Buffer_Start    // Wrap
    END IF
    ```

---

## 3. Register Mapping Table

| Guest (Intel) | Host (ARM) | Role |
| :--- | :--- | :--- |
| `EAX` | `R0` | General Purpose |
| `EBX` | `R1` | General Purpose |
| `ECX` | `R2` | Counter |
| `EDX` | `R3` | Data |
| `SI` (Source Index) | `R4` | Source Pointer |
| `DI` (Dest Index) | `R5` | Destination Pointer |
| `SP` (Stack Ptr) | `R6` | **Virtual** Stack Pointer |
| `PC` (Prog Ctr) | `R7` | **Virtual** PC |
| `FLAGS` | `R8` | Virtual Status Flags |
| - | `R13` | **Emulator's** Stack Pointer (Real HW) |
| - | `R15` | **Emulator's** PC (Real HW) |

---

## 4. Manual Index (Where to Look)
* **ARM Architecture Ref Manual (DDI 0406C):**
    * **Exception Entry Pseudo-code:** Part B, Chapter B1, Section "Exception Entry".
    * **Mode Bits:** Part B, Chapter B1, Section "Processor Modes".
    * **Instruction Details:** Part A, Chapter A8 (Alphabetical List).

* **TMS320C3x User's Guide:**
    * **Registers:** Chapter 3 (CPU Registers).
    * **Serial Port Control:** Chapter 8 (Peripherals).
```
```eof
This **Reference Guide** is designed to be your primary resource during the exam. It contains the exact answers to the most common question types. Good lu
