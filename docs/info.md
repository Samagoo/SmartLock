<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->

## How it works

**SmartLock** is an electronic lock that only opens when 4 buttons are pressed in the correct order. The code is **B4 → B2 → B3 → B4**.

The design is a **synchronous finite state machine**: 8 D flip-flops with reset store the state, and combinational logic (AND, OR, XOR and NOT gates) computes the next state and drives the 7-segment display.

### Features

- **4-key code**, with repeated keys allowed (256 possible codes).
- **Secure mode:** the display counts every key press (1, 2, 3), whether it is correct or not. The lock only reveals whether the code was right after the 4th key, so the code cannot be guessed one key at a time.
- **Lockout:** after 3 wrong codes, the lock blocks and ignores every button.
- **Master key:** only the chip reset (`rst_n`) clears the lockout. The CLOSE button does not clear the failed-attempt counter.
- A correct code clears the failed-attempt counter.

### Circuit blocks

| Block | Flip-flops | Function |
|---|---|---|
| Press detector | P1 | Makes a held button count as a single key press, even if it stays pressed for many clock cycles |
| Key counter | K1, K0 | Counts how many keys have been entered in the current attempt (0–3) |
| Error flag | M | Remembers whether any key of the current attempt was wrong |
| Open | O | Lock is open (turns on the lock LED) |
| Show error | E | Shows "E" after a wrong code |
| Failure counter | F1, F0 | Counts failed attempts; locks out at 3 |

The buttons are encoded in 2 bits: B1 = 00, B2 = 01, B3 = 10, B4 = 11.

### Truth table: "is this the right key?"

The code is stored in this table. To change the code, only this table has to change.

| K1 | K0 | Button (c1 c0) | Correct |
|---|---|---|---|
| 0 | 0 | 00 (B1) | 0 |
| 0 | 0 | 01 (B2) | 0 |
| 0 | 0 | 10 (B3) | 0 |
| 0 | 0 | 11 (B4) | **1** |
| 0 | 1 | 00 (B1) | 0 |
| 0 | 1 | 01 (B2) | **1** |
| 0 | 1 | 10 (B3) | 0 |
| 0 | 1 | 11 (B4) | 0 |
| 1 | 0 | 00 (B1) | 0 |
| 1 | 0 | 01 (B2) | 0 |
| 1 | 0 | 10 (B3) | **1** |
| 1 | 0 | 11 (B4) | 0 |
| 1 | 1 | 00 (B1) | 0 |
| 1 | 1 | 01 (B2) | 0 |
| 1 | 1 | 10 (B3) | 0 |
| 1 | 1 | 11 (B4) | **1** |

### Truth table: display

Priority: Locked out > Open > Error > key count.

| State | Display | a | b | c | d | e | f | g | Lock LED |
|---|---|---|---|---|---|---|---|---|---|
| Waiting (0 keys) | – | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |
| 1 key | 1 | 0 | 1 | 1 | 0 | 0 | 0 | 0 | 0 |
| 2 keys | 2 | 1 | 1 | 0 | 1 | 1 | 0 | 1 | 0 |
| 3 keys | 3 | 1 | 1 | 1 | 1 | 0 | 0 | 1 | 0 |
| Open | A | 1 | 1 | 1 | 0 | 1 | 1 | 1 | **1** |
| Wrong code | E | 1 | 0 | 0 | 1 | 1 | 1 | 1 | 0 |
| Locked out | b | 0 | 0 | 1 | 1 | 1 | 1 | 1 | 0 |

## How to test

1. Press **reset** (`rst_n`). The display shows **–**.
2. Press **B4, B2, B3, B4**, one key at a time. The display shows 1, 2, 3 and then **A**, and the lock LED (decimal point) turns on.
3. Press **CLOSE**. The display goes back to **–**.
4. Enter a wrong code, for example B1, B1, B1, B1. The display shows 1, 2, 3 and then **E**.
5. Enter a wrong code 2 more times. On the third failure the display shows **b** (locked out) and no button responds anymore, not even CLOSE.
6. Press **reset** (master key) to unlock.

On the Tiny Tapeout demo board, each "press" means switching the matching DIP switch (ui[0]–ui[4]) on and then off again. In the Wokwi simulation you can use the colored push buttons, or the 1, 2, 3, 4 and C keys on your keyboard. After starting the simulation, press RESET once so all flip-flops start at 0.

## External hardware

- 7-segment display (included on the Tiny Tapeout demo board).
- Optional: 4 push buttons + 1 CLOSE button with 10 kΩ pull-down resistors on ui[0]–ui[4], instead of the DIP switches.
- Optional: an LED, relay or solenoid on uo[7] to act as the lock.
