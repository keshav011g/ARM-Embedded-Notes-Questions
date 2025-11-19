# TMS320C3x DSP Exam Notes

## 1. CPU Registers (Chapter 3)

This chapter defines the core registers you manipulate in assembly.

### 1.1 Key Registers
* **R0-R7:** Extended-precision registers (40-bits). Used for integer and floating-point math.
* **AR0-AR7:** Auxiliary Registers (32-bits). Used for addressing memory (pointers) and loop counters.
* **DP:** Data Page Pointer (Used for direct addressing).
* **IR0, IR1:** Index Registers (Used for indexing addressing).
* **BK:** Block Size Register (Used for circular addressing).
* **SP:** Stack Pointer.
* **ST:** Status Register (Contains flags C, Z, N, V, GIE, etc.).
* **IE:** Interrupt Enable Register.
* **IF:** Interrupt Flag Register.

### 1.2 Status Register (ST) Flags
* **C (Bit 0):** Carry.
* **Z (Bit 1):** Zero.
* **N (Bit 2):** Negative.
* **V (Bit 3):** Overflow.
* **OVM (Bit 4):** Overflow Mode (0=Normal, 1=Saturation).
* **RM (Bit 6):** Repeat Mode flag.
* **GIE (Bit 13):** Global Interrupt Enable (1=Enable, 0=Disable).

---

## 2. Timer/Counter (Chapter 12)

The 'C3x has two 32-bit timers (Timer 0 and Timer 1). They can count internal clocks or external events.

### 2.1 Key Registers
* **TxCNT:** Timer Counter Register (Current count).
* **TxPRD:** Timer Period Register (Reload value).
* **TxGCR:** Timer Global Control Register (Configuration).

### 2.2 Timer Global Control Register (TxGCR)
*This is critical for configuration questions.*



* **FUNC (Bit 0):** Function.
    * 0 = I/O Pin (General Purpose).
    * 1 = Timer/Counter Mode.
* **H/R (Bit 6):** Hold/Run.
    * 0 = Hold (Stop).
    * 1 = Run (Start).
    * *Note: In some contexts, this bit is referred to as GO/HLD*.
* **DATIN (Bit 7):** Data Input (Read TCLK pin).
* **DATOUT (Bit 8):** Data Output (Write TCLK pin).
* **INV (Bit 9):** Invert Output.
* **CLKSRC (Bit 10):** Clock Source.
    * 1 = Internal Clock (CPU/2).
    * 0 = External Clock (TCLK pin).
* **C/P (Bit 11):** Clock/Pulse Mode.
    * 1 = Clock Mode (50% duty cycle output).
    * 0 = Pulse Mode (Pulse width = 1 cycle).

### 2.3 Potential Exam Question: Timer Setup
**Q:** Configure Timer 0 to generate a periodic interrupt every 1000 cycles using the internal clock.
**Solution Logic:**
1.  **Period:** Load `T0PRD` with 1000 (`0x3E8`).
2.  **Control:**
    * `FUNC` = 1 (Timer).
    * `CLKSRC` = 1 (Internal).
    * `H/R` = 1 (Start).
    * Write `0x6C1` (example value, verify specific bit positions) to `T0GCR`.

**Code:**
```assembly
LDI 1000, R0
STI R0, @T0PRD      ; Load Period

LDI 0x2C1, R0       ; FUNC=1, H/R=1, CLKSRC=1 (Example mask)
STI R0, @T0GCR      ; Start Timer
```

---

## 3. Serial Ports (Chapter 12)

The 'C3x has two serial ports (Port 0 and Port 1). They support standard serial communication and handshake modes.

### 3.1 Key Registers
* **SxGCR:** Global Control Register.
* **SxXCR:** Transmit Control Register.
* **SxRCR:** Receive Control Register.
* **SxXSR:** Transmit Port Control Register (Pin Mux).
* **SxRSR:** Receive Port Control Register (Pin Mux).
* **DxX:** Data Transmit Register.
* **DxR:** Data Receive Register.

### 3.2 Global Control Register (SxGCR)
* **HS (Bit 5):** Handshake Mode (0=No Handshake, 1=Handshake).
* **RCLK (Bit 6):** Receive Clock Source (0=Ext, 1=Int).
* **XCLK (Bit 7):** Transmit Clock Source (0=Ext, 1=Int).
* **RVAREN (Bit 9):** Receiver Variable Rate Enable.
* **XVAREN (Bit 10):** Transmitter Variable Rate Enable.
* **RFSM (Bit 11):** Receive Frame Sync Mode (0=Standard, 1=Burst).
* **XFSM (Bit 12):** Transmit Frame Sync Mode.
* **RLEN (Bits 22-21):** Receive Length (00=8b, 01=16b, 10=24b, 11=32b).
* **XLEN (Bits 24-23):** Transmit Length.

### 3.3 Potential Exam Question: Serial Setup
**Q:** Configure Serial Port 0 for 32-bit transfer, Internal Clock, No Handshake.
**Solution Logic:**
1.  **Registers:** Set `S0GCR`.
2.  **Bits:**
    * `RLEN` = 11 (32-bit).
    * `XLEN` = 11 (32-bit).
    * `HS` = 0 (No handshake).
    * `XCLK` / `RCLK` = 1 (Internal).

**Code:**
```assembly
; Calculate Control Word (Binary -> Hex)
; RLEN=11, XLEN=11, HS=0, CLK=1...
LDI 0x018000C0, R0  ; Example Value
STI R0, @S0GCR
```

### 3.4 Communication Example: Handshake Mode
**Q:** Write a routine to send data on Serial Port 0 only when the external device is ready (Handshake Mode).
**Code:**
```assembly
; 1. Configure Handshake (HS=1)
LDI  0x01800060, R0     ; Load Config: HS=1, XLEN=32, XCLK=1
STI  R0, @S0GCR

; 2. Enable Pins
LDI  0x111, R0          ; Enable DX, CLKX, FSX
STI  R0, @S0XCR

; 3. Transmit Loop (Wait for XRDY)
Tx_Loop:
    LDI  @S0GCR, R0     ; Read Status
    TSTB 0x2, R0        ; Check Bit 1 (XRDY)
    BZ   Tx_Loop        ; If 0, wait

    STI  R1, @DXR0      ; Send Data from R1
```

---

## 4. Direct Memory Access (DMA) Controller (Chapter 12)

The DMA controller moves data between memory and peripherals (like Serial Ports) without CPU intervention. This is faster and frees the CPU for processing.

### 4.1 Key Registers
* **DMA_GCR:** Global Control Register (`0x808000` for Ch0).
* **DMA_SRC:** Source Address Register (`0x808004`).
* **DMA_DST:** Destination Address Register (`0x808006`).
* **DMA_CTR:** Transfer Counter Register (`0x808008`).

### 4.2 DMA Global Control Register (DMA_GCR)
This register controls *how* data is moved and what triggers the move.

* **RUN (Bit 0-1):** `00`=Stop, `11`=Start.
* **DECR (Bit 2):** Decrement Destination Address.
* **INCR (Bit 3):** Increment Destination Address.
* **SYNC (Bits 6-7):** Synchronization Mode.
    * `00` = No Sync (Free run, move as fast as possible).
    * `01` = Source Sync (Wait for source ready, e.g., Serial Rx).
    * `10` = Destination Sync (Wait for dest ready, e.g., Serial Tx).
* **TC (Bit 10):** Transfer Count Interrupt Enable.
* **PRI (Bit 13):** Priority (0=Low, 1=High).

### 4.3 How DMA Works (The Process)
1.  **Set Source & Dest:** You tell the DMA *where* to read from and *where* to write to.
2.  **Set Count:** You tell it *how many* words to move.
3.  **Set Sync:** You tell it *when* to move. For Serial Receive, you sync to **RINT** (Receive Interrupt). The DMA sees the "Data Ready" signal before the CPU interrupt logic does.
4.  **Start:** Set the RUN bits.

### 4.4 Potential Exam Question: DMA Transfer
**Q:** Configure DMA Channel 0 to move 1024 words from Serial Port 0 Receive Register (`0x80804C`) to a buffer at `0x1000`. Use interrupt synchronization.
**Solution Logic:**
1.  **Source:** `0x80804C` (Fixed address, do not increment).
2.  **Dest:** `0x1000` (Buffer start, increment after each write).
3.  **Count:** `1024`.
4.  **Sync:** Source Synchronization (Wait for RINT0).

**Code:**
```assembly
; 1. Set Addresses
LDI  0x80804C, R0
STI  R0, @DMA_SRC0      ; Source = Serial DRR

LDI  0x1000, R0
STI  R0, @DMA_DST0      ; Dest = Memory Buffer

; 2. Set Count
LDI  1024, R0
STI  R0, @DMA_CTR0      ; Count = 1024

; 3. Configure Control
; RUN=11 (Start), INCR=1 (Dest+), SYNC=01 (Source Sync)
; Binary: ...0100 0000 1011 (Verify bit positions in manual!)
LDI  0x40B, R0          ; Example Config Word
STI  R0, @DMA_GCR0
```



---

## 5. Interrupts (Chapter 7)

The 'C3x supports vectored interrupts.

### 5.1 Vector Table
* Located at `0x000000` to `0x00003F`.
* **Reset:** `0x00`.
* **INT0:** `0x01`.
* **INT1:** `0x02`.
* **INT2:** `0x03`.
* **INT3:** `0x04`.
* **XINT0:** `0x05` (Serial Port 0 Xmit).
* **RINT0:** `0x06` (Serial Port 0 Recv).
* **TINT0:** `0x09` (Timer 0).

### 5.2 Key Registers
* **IE (Interrupt Enable):** Individual enable bits.
* **IF (Interrupt Flag):** Pending status.
* **ST (Status Register):** Global Interrupt Enable (`GIE` bit 13).

### 5.3 Potential Exam Question: Enable Timer Interrupt
**Q:** Write code to enable the Timer 0 Interrupt (TINT0) and the Global Interrupt Enable.
**Solution Logic:**
1.  **Bit Mask:** TINT0 corresponds to a specific bit in `IE` (e.g., bit 9).
2.  **Set IE:** OR the `IE` register with the mask.
3.  **Set GIE:** OR the `ST` register with `0x2000` (Bit 13).

**Code:**
```assembly
OR  0x200, IE       ; Enable TINT0 bit in IE (Check bit pos)
OR  0x2000, ST      ; Enable GIE in ST
```

---

## 6. Addressing Modes (Chapter 6)

Your professor emphasizes translation, so understanding TMS addressing is key.

* **Register:** `ADD R0, R1`
* **Direct:** `LDI @label, R0` (Uses DP register or lower 16 bits).
* **Indirect:** `LDI *AR0, R0` (Pointer).
* **Post-Increment:** `LDI *AR0++, R0` (Access, then AR0 += 1).
* **Pre-Decrement:** `LDI *--AR0, R0` (AR0 -= 1, then Access).
* **Circular:** `LDI *AR0++%, R0` (Requires `BK` register setup).
* **Bit-Reversed:** `LDI *AR0++B, R0` (Used for FFT).

### 6.1 Circular Buffer Logic
When using `%`, the address calculation is:
`Next_Addr = (Current_Addr + Step) MOD Block_Size`
(Hardware handles the wrapping automatically if `BK` is set).

---

## 7. Instruction Set Highlights (Appendix A)

* **Load/Store:**
    * `LDI` (Load Integer).
    * `LDF` (Load Float).
    * `STI` (Store Integer).
    * `STF` (Store Float).
* **Math:**
    * `ADDI` / `ADDF` (Add Integer/Float).
    * `SUBI` / `SUBF` (Subtract).
    * `MPYI` / `MPYF` (Multiply).
* **Parallel Instructions (The DSP Power):**
    * `MPYF3 || ADDF3`: Do a multiply and an add in ONE cycle.
    * *Exam Tip:* Used for Filters (`y = mx + c`).
    * Syntax: `MPYF3 *AR0++, *AR1++, R0 || ADDF3 R0, R2, R2`
* **Control Flow:**
    * `BR` (Branch Unconditional).
    * `Bcond` (Branch Conditional, e.g., `BNZ` - Branch Not Zero).
    * `DBNZ` (Decrement and Branch if Not Zero) - **Perfect for Loops!**
    * `CALL` / `RETS` (Subroutine Call / Return).

---

## 8. Sample Exam Question: FIR Filter Loop
**Q:** Write a TMS320C3x assembly loop to calculate one output of an FIR filter: $y = \sum (h[i] * x[i])$. Use parallel instructions.
**Code:**
```assembly
; Setup
LDI  100, RC        ; Repeat Counter (Loop N times)
LDI  @h_addr, AR0   ; AR0 -> Coefficients
LDI  @x_addr, AR1   ; AR1 -> Data
LDF  0.0, R0        ; Clear Accumulator (Result)
LDF  0.0, R2        ; Clear Temp

; Loop
RPTS RC             ; Repeat Next Instruction
MPYF3 *AR0++, *AR1++, R0 || ADDF3 R0, R2, R2

; Final Add
ADDF3 R0, R2, R0    ; Add last product
STI   R0, @y_addr   ; Store Result
```
```
```eof
I've updated the **TMS320C3x DSP Exam Notes** to include a new section on **Direct Memory Access (DMA)** with explanations and code examples, and added a specific code example for **Handshake Mode** communication in the Serial Ports section.
```
