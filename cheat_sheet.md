# ARM Exam Reference Guide (Open Book)

## Table of Contents

- [1. Hardware Mechanics "Cheat Sheet"](#1-hardware-mechanics-cheat-sheet)
    - [1.1 Exception Entry Sequence (Memorize)](#11-exception-entry-sequence-memorize)
    - [1.2 Processor Modes & Vectors](#12-processor-modes--vectors)
- [2. Emulation Templates (Recipes)](#2-emulation-templates-recipes)
    - [2.1 The Emulator Loop (Pseudo-code)](#21-the-emulator-loop-pseudo-code)
    - [2.2 Translation: Intel (CISC) to ARM (RISC)](#22-translation-intel-cisc-to-arm-risc)
    - [2.3 Translation: TMS320 (DSP) to ARM](#23-translation-tms320-dsp-to-arm)
- [3. Register Mapping Table](#3-register-mapping-table)
- [4. Manual Index (Where to Look)](#4-manual-index-where-to-look)
- [5. Sample Questions: Intel 80186 & TMS320C3x](#5-sample-questions-intel-80186--tms320c3x)
    - [5.1 Intel 80186: Register Manipulation & Interrupts](#51-intel-80186-register-manipulation--interrupts)
    - [5.2 TMS320C3x: Serial Communication & Registers](#52-tms320c3x-serial-communication--registers)
- [6. Suggested Additions](#6-suggested-additions)
    - [6.1 Pin Muxing Tables](#61-pin-muxing-tables)
    - [6.2 Memory Map Diagrams](#62-memory-map-diagrams)
    - [6.3 Status Register Flags](#63-status-register-flags)

---

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

---

## 5. Sample Questions: Intel 80186 & TMS320C3x

### 5.1 Intel 80186: Register Manipulation & Interrupts
**Q1: Timer Control Register Programming**
* **Problem:** Configure Timer 1 of an Intel 80186 to generate a 2kHz square wave. The CPU clock is 16 MHz.
* **Solution Logic:**
    * **Calculate Count:** Clock/4 = 4 MHz. 4 MHz / 2 kHz = 2000 counts. Max count register A (CMPA) = 1000 (for half period) or 2000 (full period depending on mode).
    * **Control Register (T1CON):** Set bits for Enable (`EN`), Continuous Mode (`CONT`), and Interrupt (`INT`) if needed.

**Q2: Interrupt Vector Table**
* **Problem:** An external interrupt arrives on INT0 (Type 12). Where does the CPU look for the ISR address?
* **Solution Logic:**
    * **Vector Address Calculation:** Type * 4.
    * **Address:** 12 * 4 = 48 = `0x030`.
    * **Action:** The CPU reads 4 bytes from `0000:0030` (2 bytes for IP, 2 bytes for CS) and jumps there.

### 5.2 TMS320C3x: Serial Communication & Registers
**Q1: Serial Port Configuration (Control Registers)**
* **Problem:** Configure TMS320C3x Serial Port 0 for:
    * 32-bit fixed data length.
    * Internal Transmit Clock.
    * No Handshaking.
* **Solution Logic:**
    * **Register:** `S0GCR` (Global Control Register).
    * **Bits:**
        * `RLEN` (Bits 22-21): Set to `11` (32-bit).
        * `CLKXP` (Bit 12): Set to `1` (Internal Clock).
        * `HS` (Bit 5): Set to `0` (No handshake).

**Q2: Circular Buffer Addressing**
* **Problem:** You have a buffer of size 100 words starting at `0x1000`. Write TMS assembly to read data circularly using `AR0`.
* **Solution Logic:**
    * **Registers:** `BK` (Block Size) = 100. `AR0` (Pointer) = `0x1000`.
    * **Instruction:** `LDI *AR0++%, R0`.
    * **Meaning:** `%` enables circular modification. The hardware ensures `AR0` wraps from `0x1063` back to `0x1000`.

---

## 6. Suggested Additions

### 6.1 Pin Muxing Tables

| Chip | Register | Function | Bits | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Intel 80186** | `P1CON` | Port 1 Control | 0-7 | Set bit `1` to enable alternate peripheral function (e.g. Timer Out), `0` for GPIO. |
| **TMS320C3x** | `SxRCR` | Serial Receive Control | 0, 4, 8 | Set `b0=1` (Enable Rx), `b4=1` (Enable Rx CLK), `b8=1` (Enable Rx Frame Sync) to mux pins to Serial Port. |
| **TMS320C3x** | `SxXCR` | Serial Transmit Control | 0, 4, 8 | Set `b0=1` (Enable Tx), `b4=1` (Enable Tx CLK), `b8=1` (Enable Tx Frame Sync) to mux pins to Serial Port. |

### 6.2 Memory Map Diagrams



* **Intel 80186:**
    * `0x00000 - 0x003FF`: Interrupt Vector Table (256 vectors * 4 bytes).
    * `0x00400 - ...`: RAM / User Data.
    * `... - 0xFFFFF`: Boot ROM (Reset vector at `FFFF0`).
    * *Note: Peripheral Control Block (PCB) is relocatable, typically at `FF00` in I/O space.*

* **TMS320C3x:**
    * `0x000000 - 0x0000FF`: Interrupt Vector Table (Reset at `0x00`).
    * `0x808000 - 0x809FFF`: Peripheral Bus Memory Map (Serial Ports, Timers, DMA).
        * `0x808040`: Serial Port 0 Global Control.
        * `0x809800`: Timer 0 Global Control.

### 6.3 Status Register Flags

| Architecture | Register | Flag | Bit | Function |
| :--- | :--- | :--- | :--- | :--- |
| **Intel 80x86** | `FLAGS` | **CF** | 0 | Carry Flag (Unsigned Overflow) |
| | | **PF** | 2 | Parity Flag |
| | | **AF** | 4 | Auxiliary Carry (BCD math) |
| | | **ZF** | 6 | Zero Flag (Result is zero) |
| | | **SF** | 7 | Sign Flag (Result is negative) |
| | | **TF** | 8 | Trap Flag (Single Step Debugging) |
| | | **IF** | 9 | Interrupt Enable Flag |
| | | **DF** | 10 | Direction Flag (String ops: 0=Inc, 1=Dec) |
| | | **OF** | 11 | Overflow Flag (Signed Overflow) |
| **TMS320C3x** | `ST` | **C** | 0 | Carry |
| | | **Z** | 1 | Zero |
| | | **N** | 2 | Negative |
| | | **V** | 3 | Overflow |
| | | **OVM** | 4 | Overflow Mode (Saturation enable) |
| | | **RM** | 6 | Repeat Mode Flag |
| | | **GIE** | 13 | Global Interrupt Enable |
```
```eof
