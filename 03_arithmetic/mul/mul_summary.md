# Multiplication (`mul`)

`MUL` is an unsigned multiply. The result is twice as wide as the operands, so it doesn't overflow: `AL x r/m8` goes into `AX`, `AX x r/m16` goes into `DX:AX`, and `EAX x r/m32` goes into `EDX:EAX`.

Only **CF and OF** are defined after `MUL`, and they always have the same value. Both are set when the upper half of the result (`AH`, `DX` or `EDX`) is non-zero, meaning the result doesn't fit in the lower half. Both are cleared when the upper half is zero. SF, ZF, AF and PF are undefined, so any value GDB shows for them is a leftover and I don't draw conclusions from it.

## Program 1: `mul1.asm`

Multiplies `25` by `10` using the 8-bit form `mul byte [num2]`.

### How it runs

- **`mov al, [num1]`:** `AL = 25 (0x19)`. Flags stay `[ IF ]`, because `MOV` doesn't touch them.
- **`mul byte [num2]`:** `AX = AL x 10 = 250 (0x00FA)`.

### Flags after the `mul`

- **CF cleared and OF cleared:** the result is `0x00FA`, so the upper half `AH` is `0x00`. 250 fits in `AL` alone (max 255), so the full result carries no extra information in `AH`.
- **SF, ZF, AF, PF:** undefined after `MUL`.

## Program 2: `mul2.asm`

Multiplies `3000` by `200` using the 16-bit form `mul word [num2]`.

### How it runs

- **`mov ax, [num1]`:** `AX = 3000 (0x0BB8)`. Flags stay `[ IF ]`.
- **`mul word [num2]`:** `DX:AX = AX x 200 = 600000 (0x000927C0)`, so `DX = 0x0009` and `AX = 0x27C0`.

### Flags after the `mul`

- **CF set and OF set:** 600000 is larger than 65535, so it can't fit in `AX` alone. The upper half `DX` is `9`, which is non-zero, so `MUL` sets CF and OF to say the high half is in use.
- **SF, ZF, AF, PF:** undefined after `MUL`.

CF and OF after `MUL` answer one question: did the result spill into the upper half (`AH`, `DX`, `EDX`)? In `mul1`, 250 stays in the lower half, so both are 0. In `mul2`, 600000 spills into `DX`, so both are 1. The program must read `DX` as well as `AX`, or it would see only the low 16 bits of the answer (`0x27C0 = 10176`).