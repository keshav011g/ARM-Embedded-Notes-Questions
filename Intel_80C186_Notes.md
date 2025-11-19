# Intel 80C186EB Exam Notes

## 1. Timer/Counter Unit (TCU) (Chapter 9)

This chapter is critical. Your exams frequently ask about configuring timers for specific frequencies or modes.

### 1.1 Overview
The 80C186EB has three 16-bit timers.
* **Timer 0 & Timer 1:** Can be connected to external pins (T0IN, T1IN). Can count internal or external events. Can generate output waveforms (T0OUT, T1OUT).
* **Timer 2:** Internal only. Used as a prescaler for Timer 0/1 or as a DMA request source.

### 1.2 Key Registers
You must know these addresses and bit definitions to write configuration code.

| Register | Offset | Description |
| :--- | :--- | :--- |
| **TxCNT** | `+00h` | **Timer Count Register:** Current value of the timer (increments). |
| **TxCMPA** | `+02h` | **Compare Register A:** Max count value A. Timer resets or toggles output when it matches this. |
| **TxCMPB** | `+04h` | **Compare Register B:** Max count value B (only for Timer 0/1). Used in Dual Maxcount mode. |
| **TxCON** | `+06h` | **Timer Control Register:** The "Brain". Sets mode, enable, interrupt. |

### 1.3 Programming the Control Register (TxCON)
*This is the most likely target for an exam question.*



* **EN (Bit 15):** 1 = Enable Timer.
* **INH (Bit 14):** Inhibit. Set to 1 to allow writing to other bits in this register (safety feature).
* **INT (Bit 13):** 1 = Enable Interrupt when Max Count is reached.
* **RIU (Bit 12):** Register In Use. (Read only status).
* **MC (Bit 5):** Maximum Count.
    * 0 = Timer resets to 0 and continues (Retrigger).
    * 1 = Timer stops after reaching max count.
* **RTG (Bit 4):** Retrigger.
    * 0 = Input ignored.
    * 1 = Active input transition resets timer.
* **P (Bit 3):** Prescaler.
    * 0 = Source is CPU Clock / 4.
    * 1 = Source is External Clock (Timer 2 uses TMR IN 0/1).
* **EXT (Bit 2):** External Clock.
    * 0 = Use Internal Clock.
    * 1 = Use External Clock (Timer 0/1 only).
* **ALT (Bit 1):** Alternate Mode.
    * 0 = Single Maxcount (Uses CMPA only).
    * 1 = Dual Maxcount (Uses CMPA and CMPB).
* **CONT (Bit 0):** Continuous Mode.
    * 0 = One-shot (stops after max count).
    * 1 = Continuous (resets and continues).

### 1.4 Potential Exam Question: Timer Setup
**Q:** Configure Timer 0 to generate a 1kHz square wave. The CPU clock is 16 MHz. Assume the timer block is mapped to `0xFF00` in I/O space.
**Solution Logic:**
1.  **Calc Freq:** Internal timer clock = CPU / 4 = 16MHz / 4 = 4 MHz.
2.  **Calc Count:** 1kHz Square wave means 2 transitions per cycle (High -> Low, Low -> High).
    * Period = 1ms. Half-period = 0.5ms.
    * Counts = 0.5ms / (1/4MHz) = 2000 counts.
3.  **Setup Registers:**
    * `T0CMPA` = 2000.
    * `T0CMPB` = 2000 (for symmetric wave in Dual Mode).
    * `T0CON`: EN=1, INT=1 (if needed), CONT=1, ALT=1 (Dual mode for square wave).

**Code:**
```assembly
MOV DX, 0xFF52      ; T0CMPA Address (Offset +02h)
MOV AX, 2000
OUT DX, AX

MOV DX, 0xFF54      ; T0CMPB Address (Offset +04h)
OUT DX, AX

MOV DX, 0xFF56      ; T0CON Address (Offset +06h)
MOV AX, 0C003h      ; EN=1, INT=1, CONT=1, ALT=1
OUT DX, AX
```

---

## 2. Serial Communications Unit (SCU) (Chapter 10)

Focus on **Asynchronous Mode** (UART). This is standard for "communicating with other devices."

### 2.1 Overview
The 80C186EB has two independent serial ports (Channel 0 & 1). They support Asynchronous (UART) and Synchronous modes.

### 2.2 Key Registers
* **SxSTS:** Status Register (Tx Empty, Rx Ready, Errors).
* **SxCON:** Control Register (Mode, Parity, Stop bits).
* **SxRBUF:** Receive Buffer (Read data here).
* **SxTBUF:** Transmit Buffer (Write data here).
* **B1CMP / B1CNT:** Baud Rate Generator registers.

### 2.3 Programming the Control Register (SxCON)


* **MODE (Bits 15-13):**
    * `001` = Mode 1 (Async, 10 bits: Start + 8 Data + Stop).
    * `011` = Mode 3 (Async, 11 bits: Start + 9 Data + Stop).
* **PEN (Bit 4):** Parity Enable.
* **EVN (Bit 3):** Even Parity (1=Even, 0=Odd).

### 2.4 Potential Exam Question: Serial Comm Setup
**Q:** Initialize Serial Port 0 for 9600 baud, 8 data bits, No Parity, 1 Stop bit (Mode 1). CPU Clock is 16MHz.
**Solution Logic:**
1.  **Baud Calculation:** Baud = (CPU_Clk / 2) / (Baud_Reg + 1).
    * 9600 = (8,000,000) / (B + 1)
    * B + 1 = 833.33 -> B = 832.
2.  **Control Register:** Mode 1 (Async 10-bit total frame).
    * Set `MODE` = 001.

**Code:**
```assembly
; 1. Set Baud Rate
MOV DX, 0xFF68      ; B1CMP Address (Baud Compare)
MOV AX, 832         ; Count for 9600 bps
OUT DX, AX

; 2. Configure Port Control
MOV DX, 0xFF60      ; S0CON Address
MOV AX, 02000h      ; Mode 1 (001...), PEN=0
OUT DX, AX
```

---

## 3. Interrupt Control Unit (ICU) (Chapter 8)

The exam often asks to "write the code to execute an interrupt." This implies setting up the **Interrupt Vector Table** and the **Mask Register**.

### 3.1 Key Registers
* **EOI (End of Interrupt):** Must write to this at the end of an ISR to allow new interrupts.
* **IMASK (Interrupt Mask):** 1 = Masked (Disabled), 0 = Enabled.
* **REQST (Request Status):** Shows which interrupts are pending.

### 3.2 Vector Table Logic
The 80C186 uses a vector table at `0000:0000`.
* **Vector Address = Type * 4.**
* Each entry is 4 bytes: `IP` (Instruction Pointer) then `CS` (Code Segment).

### 3.3 Potential Exam Question: Enable RFU Interrupt
**Q:** The system uses the **Receive Frame Unit (RFU)** interrupt, mapped to INT0. Write the initialization code to enable this interrupt and point it to your ISR at `My_ISR`.
**Solution Logic:**
1.  **Disable Global Interrupts:** `CLI`.
2.  **Set Vector Table:** INT0 is usually Type 12 (check manual mapping).
    * Vector Address = 12 * 4 = 48 (`0x30`).
    * Write offset of `My_ISR` to `0000:0030`.
    * Write segment of `My_ISR` to `0000:0032`.
3.  **Unmask Interrupt:** Clear the bit for INT0 in `IMASK`.
4.  **Enable Global Interrupts:** `STI`.

**Code:**
```assembly
CLI                 ; 1. Disable interrupts
XOR AX, AX
MOV DS, AX          ; Set DS to 0000h

MOV BX, 30h         ; 2. Vector Address for Type 12
MOV WORD PTR [BX], OFFSET My_ISR ; Store IP
MOV WORD PTR [BX+2], SEG My_ISR  ; Store CS

MOV DX, 0xFF28      ; 3. IMASK Register Address
IN  AX, DX
AND AX, 0FFEFh      ; Clear bit 4 (Assume INT0 is bit 4)
OUT DX, AX

STI                 ; 4. Enable interrupts
```

---

## 4. Chip-Select Unit (Chapter 6)

This relates to "Memory Map" questions. You define where ROM, RAM, and Peripherals live in memory.

### 4.1 Key Registers
* **UCS (Upper Chip Select):** Usually for Boot ROM.
* **LCS (Lower Chip Select):** Usually for RAM starting at 0.
* **MCS (Mid-Range Chip Select):** For other devices.
* **PCS (Peripheral Chip Select):** For I/O devices.

### 4.2 Potential Exam Question: Memory Map
**Q:** Configure `LCS` to enable a 64KB RAM starting at address `00000h`.
**Solution:**
* **LCS Start:** Fixed at 0.
* **LCS Stop:** Programming the **Block Size** in `LMCS` register.
* **Value:** Bits define the size. (Check manual table: typically specific bits set block size).

---

## 5. Refresh Control Unit (Chapter 11)

Rare but possible. Used for DRAM refresh.
**Q:** Enable DRAM Refresh every 15 microseconds.
**Logic:** Calculate count based on CPU clock, write to `RFTIME` register, enable `REN` bit in `RFCON`.

---

## 6. Register Manipulation & Macros (General)

Your professor asks for **Macros** to simplify tasks.

### 6.1 Macro Example: Register Setup
**Q:** Write a macro `INIT_REG(Addr, Val)` to simplify register setup.
**Code:**
```assembly
INIT_REG MACRO Addr, Val
    MOV DX, Addr
    MOV AX, Val
    OUT DX, AX
ENDM

; Usage:
INIT_REG 0xFF56, 0C003h ; Enable Timer 0
```

### 6.2 Macro Example: Wait for Serial Char
**Q:** Write a macro that waits for a character on Serial Port 0.
**Code:**
```assembly
WAIT_CHAR MACRO
    LOCAL WaitLoop
WaitLoop:
    MOV DX, 0xFF62      ; S0STS Address
    IN  AX, DX
    TEST AX, 0040h      ; Check RI (Receive Interrupt/Ready) bit
    JZ  WaitLoop        ; If 0, loop back
    MOV DX, 0xFF64      ; S0RBUF Address
    IN  AX, DX          ; Read Data
ENDM
```

---

## 7. Study Checklist

1.  **Timers:** Calculate count values for different frequencies. Memorize `TxCON` bits.
2.  **Serial:** Calculate baud rate divisors. Memorize `SxCON` bits.
3.  **Interrupts:** Memorize the Vector Table math (Type * 4) and the EOI command.
4.  **Memory:** Understand how to set Chip Selects for specific memory sizes.
```
