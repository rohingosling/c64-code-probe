# Code Probe User Manual

**Program:** Code Probe<br>
**Version:** 2.1<br>
**Release:** 1988<br>
**Last Update:** 2026<br>
**Platform:** Commodore 64<br>
**Author:** Rohin Gosling<br>

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Loading and Starting](#2-loading-and-starting)
3. [The Monitor Prompt](#3-the-monitor-prompt)
4. [Command Reference](#4-command-reference)
   - [D -- Hex Dump](#41-d--hex-dump)
   - [A -- Alter Mode](#42-a--alter-mode)
   - [R -- Display Registers](#43-r--display-registers)
   - [R reg val -- Set Register](#44-r-reg-val--set-register)
   - [RF -- Display Flags](#45-rf--display-flags)
   - [RF flag bit -- Set Flag](#46-rf-flag-bit--set-flag)
   - [F -- Fill Memory](#47-f--fill-memory)
   - [T -- Transfer Memory](#48-t--transfer-memory)
   - [G -- Execute Program](#49-g--execute-program)
   - [S -- Save File](#410-s--save-file)
   - [L -- Load File](#411-l--load-file)
   - [CLS -- Clear Screen](#412-cls--clear-screen)
   - [EXIT -- Exit to BASIC](#413-exit--exit-to-basic)
5. [Error Messages](#5-error-messages)
6. [Shadow Registers](#6-shadow-registers)
7. [The BRK Return Mechanism](#7-the-brk-return-mechanism)
8. [Tutorials](#8-tutorials)
9. [Memory Map](#9-memory-map)
10. [Quick Reference Card](#10-quick-reference-card)

---

## 1. Introduction

Code Probe is a software-based alternative to hardware cartridge monitors such as Action Replay.

The design of `Code Probe` was inspired by the DOS `DEBUG` utility, and presents a similar terminal-style user interface and commands. All numeric input is hexadecimal. Addresses are 4 digits, byte values are 2 digits, and device numbers are 2 digits.

### Features

- **Memory Inspection** - Hex dump with ASCII character display.
- **Memory Editing** - Interactive alter mode with cursor navigation and auto-advance.
- **Register Management** - View and modify all CPU registers and processor status flags.
- **Memory Operations** - Fill and transfer (copy) arbitrary blocks of memory.
- **File I/O** - Save and load PRG and SEQ files to/from disk.
- **Program Execution** - Run machine language programs with a full shadow register load and BRK-based return to the monitor.
- **Directory Listing** - List files on a connected disk device.
- **Screen Control** - Clear the display with a single command.
- **Exit to BASIC** - Exit to BASIC and return to Code Probe with `SYS 49152`.

---

## 2. Loading and Starting

Code Probe loads at address $C000 (49152 decimal) and occupies approximately
4 KB of RAM in the $C000--$CFFF region.

### From Disk

```
LOAD "CODEPROBE",8,1
SYS 49152
```

The `,8,1` parameter loads the program to its native address ($C000) rather
than the default BASIC area. After loading, `SYS 49152` transfers control to
Code Probe.

### From VICE Emulator

Use the VICE **Autostart** feature to load `codeprobe.prg` directly, or type
the `LOAD` and `SYS` commands above at the BASIC prompt.

### What Happens at Startup

1. The screen border and background are set to black.
2. Text color is set to green.
3. The screen is cleared.
4. The title banner is displayed: `CODE PROBE (2.1) - ROHIN GOSLING`
5. The BRK interrupt vector is installed (enabling return from executed
   programs).
6. Shadow registers are initialized to default values.
7. The monitor prompt appears.

---

## 3. The Monitor Prompt

The monitor prompt is a colon followed by a space:

```
: █
```

Type a command and press **RETURN** to execute it. After each command
completes, a blank line is printed followed by a new prompt.

All numeric values are entered in **hexadecimal**. Addresses are 4-digit hex
values (e.g., `1000`, `C000`, `FFFF`). Byte values are 2-digit hex values
(e.g., `00`, `FF`, `A5`).

Commands are **not** case-sensitive on the C64 (the keyboard produces
uppercase by default).

---

## 4. Command Reference

### 4.1 D -- Hex Dump

**Syntax:** `D <start> <end>`

Displays the contents of memory from `start` to `end` (inclusive). Both
addresses are 4-digit hex values.

**Output format:**

Each row shows 8 bytes:

```
  AAAA: HH HH HH HH-HH HH HH HH CCCCCCCC
```

- **AAAA** -- address of the first byte in the row.
- **HH** -- hex value of each byte, with a `-` separator between the 4th and
  5th bytes.
- **CCCCCCCC** -- character representation of the same 8 bytes (screen code
  glyphs).

If the range does not end on an 8-byte boundary, unused positions are shown
as `..`:

```
  1008: 00 00 00 ..-.. .. .. .. ...
```

A summary line follows the dump showing the total bytes displayed:

```
  XXXX (Y)
```

Where `XXXX` is the hex byte count (zero-padded to 4 digits) and `Y` is the
decimal equivalent.

**Examples:**

```
: D 1000 100F
  1000: 00 00 00 00-00 00 00 00 ........
  1008: 00 00 00 00-00 00 00 00 ........
  0010 (16)

: D 1000 1004
  1000: 00 00 00 00-00 .. .. .. .....
  0005 (5)
```

---

### 4.2 A - Alter Mode

**Syntax:** `A <address>`

Enters alter mode for writing hex bytes directly to RAM, starting at the
specified address.

**Alter mode display:**

```
  1000: █
```

The address prompt shows the current write address. You type hex digits
(0--9, A--F) to enter byte values. Invalid characters are silently ignored.

**How input works:**

- After typing two hex digits (one complete byte), the cursor automatically
  advances past a space to the next byte position.
- Up to **10 bytes** can be entered per line.
- If you enter 10 bytes, the line is committed to RAM automatically and a
  new line begins at the next address.

**Keys:**

| Key              | Action                                                |
|------------------|-------------------------------------------------------|
| 0--9, A--F       | Enter a hex digit.                                    |
| RETURN           | Commit the current line to RAM and advance to the     |
|                  | next line. On an empty line, exits alter mode.        |
| Cursor Left      | Move one hex digit to the left, skipping spaces.      |
|                  | Stops at the first digit of the first byte.           |
| Cursor Right     | Move one hex digit to the right, skipping spaces.     |
|                  | Cannot advance past the current entry point.          |
| DEL (INST/DEL)   | Delete the last nibble entered. Only works at the     |
|                  | entry frontier (the rightmost position).              |

**Exiting alter mode:**

Press **RETURN** on an empty line. A byte count summary is displayed:

```
  XXXX (Y)
```

**Example session:**

```
: A 1000
  1000: A9 FF 00
  1003:
  0003 (3)

: D 1000 1002
  1000: A9 FF 00 ..-.. .. .. .. ...
  0003 (3)
```

---

### 4.3 R - Display Registers

**Syntax:** `R`

Displays the current values of all shadow registers:

```
A:00 X:00 Y:00 SP:00 PC:0000 P:20 IO:37
```

| Register | Size    | Description                                        |
|----------|---------|----------------------------------------------------|
| A        | 8-bit   | Accumulator.                                       |
| X        | 8-bit   | X index register.                                  |
| Y        | 8-bit   | Y index register.                                  |
| SP       | 8-bit   | Stack pointer.                                     |
| PC       | 16-bit  | Program counter.                                   |
| P        | 8-bit   | Processor status register (default $20).           |
| IO       | 8-bit   | Processor port at address $01 (memory bank config).|

---

### 4.4 R reg val - Set Register

**Syntax:** `R <register> <value>`

Sets a shadow register to the specified value and displays the updated
register line.

Code Probe maintains its own copy of the CPU registers in RAM, called
**shadow registers**. The `R` command modifies these shadow copies, not the
live CPU registers. When you execute a program with `G`, the shadow values
are loaded into the real CPU. When the program returns via BRK, the real CPU
state is captured back into the shadow registers. See
[Section 6](#6-shadow-registers) for a full explanation.

- 8-bit registers (`A`, `X`, `Y`, `SP`, `P`, `IO`) require a 2-digit hex
  value.
- The 16-bit register (`PC`) requires a 4-digit hex value.

**Examples:**

```
: R A FF
A:FF X:00 Y:00 SP:00 PC:0000 P:20 IO:37

: R PC C000
A:FF X:00 Y:00 SP:00 PC:C000 P:20 IO:37
```

---

### 4.5 RF - Display Flags

**Syntax:** `RF`

Displays the register line followed by the processor status flags expanded as
individual bits:

```
A:00 X:00 Y:00 SP:00 PC:0000 P:20 IO:37

  NV-BDIZC
P:00100000
```

The flag positions from bit 7 to bit 0 are:

| Bit | Flag | Name             |
|-----|------|------------------|
| 7   | N    | Negative         |
| 6   | V    | Overflow         |
| 5   | -    | (unused, always 1)|
| 4   | B    | Break            |
| 3   | D    | Decimal          |
| 2   | I    | Interrupt disable|
| 1   | Z    | Zero             |
| 0   | C    | Carry            |

---

### 4.6 RF flag bit - Set Flag

**Syntax:** `RF <flag> <bit>`

Sets an individual processor status flag to 0 or 1 and displays the updated
register and flag view. Like the `R` command, this modifies the **shadow
registers** rather than the live CPU. The updated flag value will take effect
the next time you run a program with `G`. See
[Section 6](#6-shadow-registers) for details.

- `flag` is one of: `N`, `V`, `-`, `B`, `D`, `I`, `Z`, `C`.
- `bit` must be `0` or `1`.

**Example:**

```
: RF C 1
A:00 X:00 Y:00 SP:00 PC:0000 P:21 IO:37

  NV-BDIZC
P:00100001
```

---

### 4.7 F - Fill Memory

**Syntax:** `F <address> <count> <value>`

Fills `count` bytes of memory starting at `address` with the byte `value`.

- `address` and `count` are 4-digit hex values.
- `value` is a 2-digit hex value.
- If the fill range would exceed $FFFF, the operation is truncated to the
  boundary and an error is printed.

**Examples:**

```
: F 1000 0100 00
```

Fills 256 bytes at $1000--$10FF with zeros.

```
: F 1000 0010 EA
```

Fills 16 bytes at $1000--$100F with $EA (the NOP opcode).

---

### 4.8 T - Transfer Memory

**Syntax:** `T <source> <count> <destination>`

Copies `count` bytes from `source` to `destination`.

- All parameters are 4-digit hex values.
- The source memory is **not** cleared after the copy (this is a copy, not a
  move).
- If either the source or destination range would exceed $FFFF, the operation
  is truncated and an error is printed.

**Example:**

```
: F 1000 0010 AA
: T 1000 0010 2000
: D 2000 200F
  2000: AA AA AA AA-AA AA AA AA ........
  2008: AA AA AA AA-AA AA AA AA ........
  0010 (16)
```

---

### 4.9 G - Execute Program

**Syntax:** `G <address>`

Begins executing machine code at the specified address.

Before the jump, the shadow register values are loaded into the real CPU
registers. The program runs with the exact register state shown by the `R`
command.

Programs can return control to Code Probe by executing a **BRK** instruction
(opcode $00). When BRK is encountered, all CPU registers are captured back
into the shadow registers and the monitor prompt reappears.

If the program does not execute BRK (e.g., it enters an infinite loop or
jumps to the KERNAL warm start), control does not return to Code Probe.

**Example:**

```
: A 1000
  1000: A9 41 20 D2 FF 00
  1006:
  0006 (6)

: G 1000
A
A:41 X:00 Y:00 SP:00 PC:1005 P:20 IO:37
```

This program loads the PETSCII code for `A` into the accumulator, calls the
KERNAL CHROUT routine to print it, then executes BRK to return.

---

### 4.10 S - Save File

**Syntax:** `S "<filename>" <device> <start> <end> [<load_address>]`

Saves memory from `start` to `end` (inclusive) to a file on the specified
device.

- `filename` is enclosed in double quotes.
- `device` is a 2-digit hex device number (typically `08` for disk drive).
- `start` and `end` are 4-digit hex addresses.
- `load_address` (optional) is a 4-digit hex value.

**File types:**

- If `load_address` is **omitted**, the file is saved as a sequential (SEQ)
  file with no header. All bytes from `start` to `end` are written as raw
  data.
- If `load_address` is **provided**, the file is saved as a program (PRG)
  file. A 2-byte header containing the load address is written first,
  followed by the data bytes.

**Output on success:**

```
  BYTES SAVED: XXXX (Y)
  ADDRESS:     XXXX (Y)       <-- only if load_address was specified
```

**Examples:**

Save as PRG (with load address):

```
: S "MYPROG" 08 1000 10FF 1000
  BYTES SAVED: 0100 (256)
  ADDRESS:     1000 (4096)
```

Save as SEQ (raw data, no header):

```
: S "MYDATA" 08 2000 20FF
  BYTES SAVED: 0100 (256)
```

---

### 4.11 L - Load File

Code Probe supports three modes depending on the number of arguments.

#### Directory Listing

**Syntax:** `L <device>`

Lists all files on the disk in the specified device. The device number is a
2-digit hex value (e.g., `08` for device 8).

**Output format:**

Each file is shown on its own line with the filename padded to 16 characters,
followed by the file type prefixed with a period:

```
  FILENAME        .PRG
```

A summary line follows showing the total file count:

```
  XXXX (Y)
```

**Example:**

```
: L 08
  MYPROG          .PRG
  MYDATA          .SEQ
  0002 (2)
```

#### PRG Load

**Syntax:** `L "<filename>" <device>`

Loads a program file from disk. The first two bytes of the file are read as a
load address header, and the remaining data is loaded into RAM at that
address.

Use this mode for files saved with a load address (the PRG format).

**Output on success:**

```
  BYTES LOADED: XXXX (Y)
  ADDRESS:      XXXX (Y)
```

**Example:**

```
: L "MYPROG" 08
  BYTES LOADED: 0100 (256)
  ADDRESS:      1000 (4096)
```

#### SEQ / Raw Load

**Syntax:** `L "<filename>" <device> <address>`

Loads a file sequentially, storing all bytes starting at the specified
address. No header bytes are consumed from the file.

Use this mode for files saved without a load address (the SEQ format).

**Example:**

```
: L "MYDATA" 08 2000
  BYTES LOADED: 0100 (256)
  ADDRESS:      2000 (8192)
```

---

### 4.12 CLS - Clear Screen

**Syntax:** `CLS`

Clears the screen and returns the cursor to the top of the display. The
monitor prompt appears at the top of a blank screen.

**Example:**

```
: CLS
```

---

### 4.13 EXIT - Exit to BASIC

**Syntax:** `EXIT`

Exits Code Probe and returns to the BASIC `READY.` prompt. Code Probe
remains in memory at $C000 and can be re-entered at any time with
`SYS 49152`.

When you re-enter, Code Probe performs a full startup: colors are reset,
the screen is cleared, shadow registers are re-initialized, and the BRK
vector is re-installed.

**Example:**

```
: EXIT

READY.
```

To return to Code Probe:

```
SYS 49152
```

---

## 5. Error Messages

| Message                    | Cause                                        |
|----------------------------|----------------------------------------------|
| `ERROR: UNKNOWN COMMAND`   | The command is not recognized.               |
| `ERROR: ILLEGAL VALUE`     | A non-hex character was entered, or a hex    |
|                            | value has the wrong width for the target.    |
| `ERROR: RAM OVERFLOW`      | A memory operation would exceed $FFFF. The   |
|                            | operation is truncated to the boundary.      |
| `ERROR: FILE NOT FOUND.`   | The specified file could not be found on the |
|                            | device.                                      |
| `ERROR: SAVE FAILED`       | The file save operation did not complete     |
|                            | successfully.                                |

---

## 6. Shadow Registers

Code Probe maintains a set of **shadow registers** in RAM that represent the
user's view of the CPU state. These are not the live CPU registers (Code
Probe itself uses the CPU to run).

| Register | Default | Description                                      |
|----------|---------|--------------------------------------------------|
| A        | $00     | Accumulator.                                     |
| X        | $00     | X index register.                                |
| Y        | $00     | Y index register.                                |
| SP       | $00     | Stack pointer.                                   |
| PC       | $0000   | Program counter.                                 |
| P        | $20     | Processor status (bit 5 always set).             |
| IO       | varies  | Processor port at $01 (read from hardware at     |
|          |         | startup).                                        |

When you execute a program with the `G` command, these shadow values are
loaded into the real CPU registers before the jump. When the program returns
via BRK, the CPU registers are captured back into the shadow registers.

You can view the shadow registers with `R`, modify them with `R <reg> <val>`,
and inspect individual processor flags with `RF`.

### Worked Example: Observing the Shadow Register Round Trip

This experiment demonstrates how shadow registers flow into the CPU and back.

**Step 1 -- Enter a small test program at $1000.**

The program loads the value $42 into the accumulator and $07 into the X
register, then returns to the monitor via BRK:

```
: A 1000
  1000: A9 42 A2 07 00
  1005:
  0005 (5)
```

| Hex       | Assembly   | Description                    |
|-----------|------------|--------------------------------|
| `A9 42`   | `LDA #$42` | Load $42 into the accumulator. |
| `A2 07`   | `LDX #$07` | Load $07 into the X register.  |
| `00`      | `BRK`      | Return to Code Probe.          |

**Step 2 -- Pre-load the shadow accumulator before execution.**

Set the shadow A register to $FF so you can see it change:

```
: R A FF
A:FF X:00 Y:00 SP:00 PC:0000 P:20 IO:37
```

**Step 3 -- Execute the program.**

```
: G 1000
A:42 X:07 Y:00 SP:00 PC:1004 P:20 IO:37
```

When the program hits BRK, the real CPU registers are captured back into the
shadow registers. Notice that A is now $42 (set by `LDA #$42`), X is $07
(set by `LDX #$07`), and PC is $1004 (the address of the BRK instruction).

**Step 4 -- Verify the shadow state persists.**

```
: R
A:42 X:07 Y:00 SP:00 PC:1004 P:20 IO:37
```

The shadow registers still hold the values captured at the breakpoint. You
can inspect them, modify them with `R`, and even re-execute the program with
`G 1000` -- the shadow values will be loaded into the CPU again at the start
of each run.

---

## 7. The BRK Return Mechanism

Code Probe installs a custom interrupt handler for the BRK instruction at
startup. This is what allows programs executed with `G` to return to the
monitor.

**How it works:**

1. At startup, Code Probe saves the original BRK vector at $0316/$0317 and
   replaces it with its own handler address.
2. When a program executes BRK (opcode $00), the 6502 pushes the program
   counter and processor status onto the stack, then jumps through the BRK
   vector.
3. Code Probe's handler captures all register values into the shadow
   registers, restores the screen colors, and returns to the monitor prompt.

**Using BRK as a breakpoint:**

You can use BRK as a simple breakpoint by writing $00 at a location of
interest in your program. When execution reaches that address, control
returns to Code Probe with all registers captured. Use `R` to inspect the
CPU state at the break point.

**Example - setting a breakpoint:**

```
: A 1005
  1005: 00
  1006:
  0001 (1)

: G 1000
A:41 X:00 Y:00 SP:00 PC:1005 P:20 IO:37
```

---

## 8. Tutorials

### 8.1 Inspecting Memory

To examine the contents of memory at any address range, use the `D` command:

```
: D C000 C01F
```

This displays 32 bytes starting at $C000 in both hex and character form.

### 8.2 Entering a Machine Language Program

Use the `A` command to enter bytes into RAM:

```
: A 1000
  1000: A9 41 20 D2 FF 00
  1006:
  0006 (6)
```

This enters a short program that prints the letter "A" and returns to the
monitor. The bytes are:

| Hex          | Assembly        | Description              |
|--------------|-----------------|--------------------------|
| `A9 41`      | `LDA #$41`      | Load PETSCII 'A'.        |
| `20 D2 FF`   | `JSR $FFD2`     | Call KERNAL CHROUT.       |
| `00`         | `BRK`           | Return to Code Probe.    |

Verify the program with `D`:

```
: D 1000 1005
  1000: A9 41 20 D2-FF 00 .. .. .A. ...
  0006 (6)
```

Execute it with `G`:

```
: G 1000
A
A:41 X:00 Y:00 SP:00 PC:1005 P:20 IO:37
```

### 8.3 Saving and Reloading a Program

After entering a program, save it to disk as a PRG file:

```
: S "PRINTA" 08 1000 1005 1000
  BYTES SAVED: 0006 (6)
  ADDRESS:     1000 (4096)
```

To verify the round trip, clear the memory region, reload, and execute:

```
: F 1000 0010 00
: L "PRINTA" 08
  BYTES LOADED: 0006 (6)
  ADDRESS:      1000 (4096)

: G 1000
A
A:41 X:00 Y:00 SP:00 PC:1005 P:20 IO:37
```

### 8.4 "Hello World" Program

This program prints "HELLO WORLD!" and waits for a key press before
returning to the monitor via BRK.

**Program listing:**

The program consists of 19 bytes of machine code followed by 15 bytes of
string data, for a total of 34 bytes at $1000--$1021.

Machine code at $1000:

| Address | Hex        | Assembly        | Description                    |
|---------|------------|-----------------|--------------------------------|
| $1000   | `A2 00`    | `LDX #$00`      | Initialize string index.       |
| $1002   | `BD 13 10` | `LDA $1013,X`   | Load next character of string. |
| $1005   | `F0 06`    | `BEQ $100D`     | Branch to key-wait on NUL.     |
| $1007   | `20 D2 FF` | `JSR $FFD2`     | Call KERNAL CHROUT.            |
| $100A   | `E8`       | `INX`           | Advance string index.          |
| $100B   | `D0 F5`    | `BNE $1002`     | Loop to next character.        |
| $100D   | `20 E4 FF` | `JSR $FFE4`     | Call KERNAL GETIN.             |
| $1010   | `F0 FB`    | `BEQ $100D`     | Loop until a key is pressed.   |
| $1012   | `00`       | `BRK`           | Return to Code Probe.          |

String data at $1013:

| Address | Hex                   | Description                           |
|---------|-----------------------|---------------------------------------|
| $1013   | `0D`                  | CR (carriage return).                 |
| $1014   | `48 45 4C 4C 4F 20`   | ASCII "HELLO ".                       |
| $101A   | `57 4F 52 4C 44 21`   | ASCII "WORLD!".                       |
| $1020   | `0D`                  | CR (carriage return).                 |
| $1021   | `00`                  | NUL terminator.                       |

The code indexes through the string at $1013, printing each byte via
CHROUT until it encounters the NUL terminator. It then polls GETIN
repeatedly until a key is pressed, and finally executes BRK to return to
the Code Probe prompt.

**Step 1 -- Enter the program.**

```
: A 1000
  1000: A2 00 BD 13 10 F0 06 20 D2 FF E8
  100B: D0 F5 20 E4 FF F0 FB 00 0D 48 45
  1016: 4C 4C 4F 20 57 4F 52 4C 44 21 0D
  1021: 00
  1022:
  0022 (34)
```

**Step 2 -- Save and run.**

```
: S "HELLO" 08 1000 1021 1000
  BYTES SAVED: 0022 (34)
  ADDRESS:     1000 (4096)

: G 1000

HELLO WORLD!

:
```

### 8.5 Copying and Filling Memory

Fill a region with a pattern byte:

```
: F 2000 0020 AA
```

Copy 32 bytes from $2000 to $3000:

```
: T 2000 0020 3000
```

Verify both regions:

```
: D 2000 201F
  2000: AA AA AA AA-AA AA AA AA ........
  2008: AA AA AA AA-AA AA AA AA ........
  2010: AA AA AA AA-AA AA AA AA ........
  2018: AA AA AA AA-AA AA AA AA ........
  0020 (32)

: D 3000 301F
  3000: AA AA AA AA-AA AA AA AA ........
  3008: AA AA AA AA-AA AA AA AA ........
  3010: AA AA AA AA-AA AA AA AA ........
  3018: AA AA AA AA-AA AA AA AA ........
  0020 (32)
```

### 8.6 Working With Registers

View the current register state:

```
: R
A:00 X:00 Y:00 SP:00 PC:0000 P:20 IO:37
```

Set the accumulator to $FF and view the flags:

```
: R A FF
A:FF X:00 Y:00 SP:00 PC:0000 P:20 IO:37

: RF
A:FF X:00 Y:00 SP:00 PC:0000 P:20 IO:37

  NV-BDIZC
P:00100000
```

Set the carry flag:

```
: RF C 1
A:FF X:00 Y:00 SP:00 PC:0000 P:21 IO:37

  NV-BDIZC
P:00100001
```

### 8.7 Writing a Program With a BASIC Stub

The previous tutorials all loaded programs at $1000 and returned to Code
Probe via BRK. This tutorial shows how to write a program that loads at
$0801 (the standard BASIC start address) with a BASIC stub, so it can be
loaded and run from BASIC like any other program:

```
LOAD "CLS",8,1
RUN
```

The program clears the screen and sets the display colors: black border,
black background, and green text.

**Program listing:**

The program consists of a 12-byte BASIC stub followed by 19 bytes of machine
code, for a total of 31 bytes at $0801--$081F.

BASIC stub -- equivalent to `10 SYS 2061`:

| Address | Hex                | Description                          |
|---------|--------------------|--------------------------------------|
| $0801   | `0B 08`            | Next-line pointer ($080B).           |
| $0803   | `0A 00`            | Line number (10).                    |
| $0805   | `9E`               | BASIC `SYS` token.                   |
| $0806   | `32 30 36 31`      | ASCII "2061" (address $080D).        |
| $080A   | `00`               | End-of-line null.                    |
| $080B   | `00 00`            | End-of-program null pointer.         |

Machine code at $080D:

| Address | Hex          | Assembly        | Description                   |
|---------|--------------|-----------------|-------------------------------|
| $080D   | `A9 00`      | `LDA #$00`      | Black (color 0).              |
| $080F   | `8D 20 D0`   | `STA $D020`     | Set border color.             |
| $0812   | `8D 21 D0`   | `STA $D021`     | Set background color.         |
| $0815   | `A9 05`      | `LDA #$05`      | Green (color 5).              |
| $0817   | `8D 86 02`   | `STA $0286`     | Set cursor/text color.        |
| $081A   | `A9 93`      | `LDA #$93`      | Clear-screen control code.    |
| $081C   | `20 D2 FF`   | `JSR $FFD2`     | Call KERNAL CHROUT.           |
| $081F   | `60`         | `RTS`           | Return to BASIC.              |

The code stores black into the VIC-II border and background registers, sets
the active text color to green, then prints the clear-screen character
via CHROUT. `RTS` returns to BASIC, which prints the `READY.` prompt in the
new text color.

**Step 1 -- Enter the program.**

```
: A 0801
  0801: 0B 08 0A 00 9E 32 30 36 31 00
  080B: 00 00 A9 00 8D 20 D0 8D 21 D0
  0815: A9 05 8D 86 02 A9 93 20 D2 FF
  081F: 60
  0820:
  001F (31)
```

Each of the first three lines enters 10 bytes and auto-commits. The fourth
line enters the final `RTS` byte; press **RETURN** to commit, then
**RETURN** again on the empty line to exit alter mode.

**Step 2 -- Verify the program.**

```
: D 0801 081F
  0801: 0B 08 0A 00-9E 32 30 36 ........
  0809: 31 00 00 00-A9 00 8D 20 ........
  0811: D0 8D 21 D0-A9 05 8D 86 ........
  0819: 02 A9 93 20-D2 FF 60 .. ........
  001F (31)
```

Compare the hex bytes against the listing above. If any byte is wrong, use
`A` at that address to correct it.

**Step 3 -- Save to disk.**

Save as a PRG file with a $0801 load address:

```
: S "CLS" 08 0801 081F 0801
  BYTES SAVED: 001F (31)
  ADDRESS:     0801 (2049)
```

The five arguments are the filename, device number, start address, end
address, and load address. The load address ($0801) is written as a 2-byte
PRG header so that `LOAD "CLS",8,1` places the data at the correct address.

**Step 4 -- Test the program.**

Exit Code Probe and run the program from BASIC:

```
: EXIT

READY.
LOAD "CLS",8,1
RUN
```

The screen clears to black with a green `READY.` prompt and cursor.
The program has done its job and can be overwritten in memory -- it only
needs to run once.

---

## 9. Memory Map

Code Probe resides in the $C000--$CFFF region. The following map shows how
it fits into the C64 memory layout:

```
       ┌───────────────────────────────┐
$FFFF  │ KERNAL ROM                    │
       │                               │
$E000  ├───────────────────────────────┤
$DFFF  │ I/O Registers (VIC, SID, CIA) │
       │                               │
$D000  ├───────────────────────────────┤
$CFFF  │ CODE PROBE ($C000-$CFFF)      │
       │                               │
$C000  ├───────────────────────────────┤
$BFFF  │ BASIC ROM                     │
       │                               │
$A000  ├───────────────────────────────┤
       │                               │
       │ Free RAM                      │
       │ (user programs and data)      │
       │                               │
$0800  ├───────────────────────────────┤
       │ Screen Memory                 │
$0400  ├───────────────────────────────┤
       │ System (zero page, stack)     │
$0000  └───────────────────────────────┘
```

**Safe areas for user programs:** $0800--$9FFF (approximately 38 KB of free
RAM when BASIC programs are not present). Avoid writing to $C000--$CFFF
(Code Probe), $D000--$DFFF (I/O), and $E000--$FFFF (KERNAL) unless you
know what you are doing.

---

## 10. Quick Reference Card

```
┌──────┬───────────────────────────────────────────────────┐
│ Cmd  │ Syntax                                            │
├──────┼───────────────────────────────────────────────────┤
│  D   │ D <start> <end>                                   │
│      │ Hex dump memory from start to end (inclusive).    │
├──────┼───────────────────────────────────────────────────┤
│  A   │ A <address>                                       │
│      │ Enter alter mode to write hex bytes to RAM.       │
├──────┼───────────────────────────────────────────────────┤
│  R   │ R                                                 │
│      │ Display all shadow registers.                     │
├──────┼───────────────────────────────────────────────────┤
│  R   │ R <register> <value>                              │
│      │ Set a shadow register.                            │
├──────┼───────────────────────────────────────────────────┤
│  RF  │ RF                                                │
│      │ Display registers with expanded flag bits.        │
├──────┼───────────────────────────────────────────────────┤
│  RF  │ RF <flag> <0|1>                                   │
│      │ Set an individual processor status flag.          │
├──────┼───────────────────────────────────────────────────┤
│  F   │ F <address> <count> <value>                       │
│      │ Fill memory with a byte value.                    │
├──────┼───────────────────────────────────────────────────┤
│  T   │ T <source> <count> <destination>                  │
│      │ Copy memory from source to destination.           │
├──────┼───────────────────────────────────────────────────┤
│  G   │ G <address>                                       │
│      │ Execute machine code at address.                  │
├──────┼───────────────────────────────────────────────────┤
│  S   │ S "<file>" <dev> <start> <end> [<load_addr>]      │
│      │ Save memory to file. With load_addr = PRG file.   │
├──────┼───────────────────────────────────────────────────┤
│  L   │ L <dev>                                           │
│      │ List files on device.                             │
├──────┼───────────────────────────────────────────────────┤
│  L   │ L "<file>" <dev>                                  │
│      │ Load PRG file (uses file's load address).         │
├──────┼───────────────────────────────────────────────────┤
│  L   │ L "<file>" <dev> <address>                        │
│      │ Load SEQ file to specified address.               │
├──────┼───────────────────────────────────────────────────┤
│ CLS  │ CLS                                               │
│      │ Clear the screen.                                 │
├──────┼───────────────────────────────────────────────────┤
│ EXIT │ EXIT                                              │
│      │ Exit to BASIC. Re-enter with SYS 49152.           │
└──────┴───────────────────────────────────────────────────┘
```

All address and count values are hexadecimal. Addresses are 4 digits, byte
values are 2 digits, device numbers are 2 digits (typically `08` for disk).
