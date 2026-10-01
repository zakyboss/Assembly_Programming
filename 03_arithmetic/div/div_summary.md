# Division (`div`)

`DIV` is an unsigned divide. Unlike `ADD` and `SUB`, **all the arithmetic flags (CF, OF, SF, ZF, AF, PF) are undefined after `DIV`**. The CPU makes no promise about them, so any value GDB shows is a leftover from earlier instructions, not information about the division. What matters is where the quotient and remainder end up. If the divisor is 0, or the quotient is too big for the destination register, the CPU raises a divide error (#DE) instead of setting a flag.

## Program 1: `div1.asm`

Divides `100` by `7` using the 8-bit form `div bl`.

### How it runs

- **`mov ax, [dividend]`:** `AX = 100 (0x0064)`. The 8-bit form divides all of `AX`.
- **`mov bl, [divisor]`:** `BL = 7`.
- **`div bl`:** `AX / BL`. The quotient goes to `AL` and the remainder to `AH`, so `AL = 14 (0x0E)` and `AH = 2`.

### Flags

Before `div`, the flags are `[ IF ]`, because the `mov` instructions don't change them. After `div`, GDB shows `[ IF ]`, which is just the old state carried over. I can't read anything into CF, ZF, SF, OF, PF or AF here because `DIV` leaves them undefined. IF is unaffected by arithmetic.

### Why it works

100 = 14 x 7 + 2. The quotient 14 fits in `AL` (max 255), so there is no #DE. If the quotient were above 255, for example 1000 / 2, the CPU would raise a divide error instead of finishing.

## Program 2: `div2.asm`

Divides `50000` by `300` using the 16-bit form `div bx`.

### How it runs

- **`mov ax, [dividend]`:** `AX = 50000 (0xC350)`, the low half of the dividend.
- **`mov dx, [highpart]`:** `DX = 0`, the high half. The 16-bit form divides the 32-bit value `DX:AX`, so `DX` must be set before `div` or the dividend would be wrong.
- **`mov bx, [divisor]`:** `BX = 300 (0x012C)`.
- **`div bx`:** `DX:AX / BX`. The quotient goes to `AX` and the remainder to `DX`, so `AX = 166 (0x00A6)` and `DX = 200 (0x00C8)`.

### Flags

As in `div1`, the flags before `div` are `[ IF ]` and the `mov` instructions leave them unchanged. After `div`, GDB shows `[ IF ]`, which is the old state and not a result of the division. The arithmetic flags are undefined after `DIV`.

### Why it works

50000 = 166 x 300 + 200. The quotient 166 fits in 16 bits, so there is no #DE. The remainder is always smaller than the divisor, which is why 200 is less than 300.





