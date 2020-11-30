# Custom Instruction Set Simulator

## Typical Instruction Structure

- 10-bit instruction is divided into:

    Opcode | Destination Register | Select Bit | Source Operand
    -|-|-|-
    4 bits | 2 bits | 1 bit | 3 bits
    
    ----

- Example: Adding R1 register to R2 and place the result in R1
    - In Assembly:

    > ```AddR``` R1, R2

    - In Machine Code:
    
    > 0000 01 0 10X

    where X can be 0 or 1 (Unused bit because R2 occupies 2 bits only)


---
    
## Opcode

- Opcode occupies 4 bits, so the instruction set can support up to 16 instructions (or operations).

    Instruction | Opcode | Examples
    -|-|-
    ```AddR```  | 0000 | ```AddR``` R1, R2 or ```AddR``` R2, 3
    ```SubR```  | 0001 | ```SubR``` R2, R1 or ```SubR``` R1, 4
    ```MultR``` | 0010 | ```MultR``` R1, R2 or ```MultR``` R2, 2
    ```DivR```  | 0011 | ```DivR``` R2, R1 or ```DivR``` R1, 5
    ```MovR```	| 0100 | ```MovR``` R1, R2 or ```MovR``` R1, 3
    ```AndR```	| 0101 | ```AndR``` R1, R2 or ```AndR``` R1, 4
    ```ORR```   | 0110 | ```ORR``` R1, R2 or ```ORR``` R2, 6
    ```Shl```   | 0111 | ```Shl``` R1, R2 or ```Shl``` R2, 1
    ```J```     | 1000 | ```J``` 6
    ```St```    | 1001 | ```St``` R1, 4
    ```Ld```    | 1010 | ```Ld``` R2, 3
    Reserved    | 1011 | -
    Reserved    | 1100 | -
    Reserved    | 1101 | -
    Reserved    | 1110 | -
    Reserved    | 1111 | -

- Reserved insructions are reserved for future work.

---

## Destination Register

- As the name implies, the destination can only be a register from the register file (no direct memory access support).

- Destination register can either be R1 or R2

- Hardware special registers like PC can't be used as the destination register (they are reserved for hardware only).

---

## Register file contents:

- Since registers occupy 2 bits, there are only 4 registers in the register file 

    Register | Equivalent Machine Code (Address)
    -|-
    RTL | 00
    R1  | 01
    R2  | 10
    PC  | 11 

---

## Select Bit

- The instruction set supports two types of instructions.

1. Instruction with a destination register and a source register.
2. Instruction with a destination register and an immediate value.

- The select bit is used to choose the type of the source operand:

    Select Bit | Source Operand
    -|-
    0 | Source register (2 bits) (either R0 or R1)
    1 | Immediate value (3 bits) (0-7)

---

## Source Operand

- It's a 3-bit code in the machine instruction

- Source operand is determined by the select bit

- When the source operand is source register, the last bit in the machine instruction is X (don't care), it can be either 0 or 1 and the instruction will be the same

    - For example: AddR R1, R2 will be 0000 01 0 10X
    - It can be 0000010100  (here X = 0)
    - Or it can be 0000010101 (here X = 1)
    - Since the source operand is a register not an immediate value, we don't care about the last bit because the source register bits are the first two bits from the 3 bits of operand

- Source register can either be R1 or R2

- Hardware special registers like PC can't be used as the source register (they are reserved for hardware only).

- If the source operand is an immediate value, all 3 bits are used and the value can ranges from 0 to 7 (8 values = 2^(3 bits) )


- There are opcodes that support both cases of source operand and others are not:

    Opcode | Source Register | Immediate Value
    -|-|-
    ```AddR```      | Supported     | Supported
    ```SubR```      | Supported     | Supported
    ```MultR```     | Supported     | Supported
    ```DivR```      | Supported     | Supported
    ```MovR```	    | Supported     | Supported
    ```AndR```	    | Supported     | Supported
    ```ORR```		| Supported     | Supported
    ```Shl```       | Supported     | Supported
    ```J```         | Not Supported | Supported
    ```St```        | Not Supported | Supported
    ```Ld```        | Not Supported | Supported

---

## Examples:

Instruction | Assembly Meaning | Machine Code | Machine Code Meaning
-|-|-|-
```AddR``` R1, R2   | R1 = R1 + R2  | 0000010100  | 0000 01 0 10X
```AddR``` R2, 4    | R2 = R2 + 4   | 0000101100  | 0000 10 1 100
```SubR``` R2, 3    | R2 = R2 - 3   | 0001101011  | 0001 10 1 011
```SubR``` R2, R1   | R2 = R2 - R1  | 0001100011  | 0001 10 0 01X
```MultR``` R1, R2  | R1 = R1 * R2  | 0010010100  | 0010 01 0 10X
```MultR``` R2, 4   | R2 = R2 * 4   | 0010101100  | 0010 10 1 100
```DivR``` R1, 5    | R1 = R1 / 5   | 0011011101  | 0011 01 1 101
```DivR``` R2, R1   | R2 = R2 / R1  | 0011100010  | 0011 10 0 01X
```MovR``` R1, 6    | R1 = 6        | 0100011110  | 0100 01 1 110
```MovR``` R1, R2   | R1 = R2       | 0100010101  | 0100 01 0 10X
```AndR``` R1, R2   | R1 = R1 & R2  | 0101010100  | 0101 01 0 10X
```AndR``` R2, 4    | R2 = R2 & 4   | 0101101100  | 0101 10 1 100
```ORR``` R1, R2    | R1 = R1 \| R2 | 0110010101  | 0110 01 0 10X
```ORR``` R2, 5     | R2 = R2 \| 5  | 0110101101  | 0110 10 1 101
```Shl``` R1, 3     | R1 = R1 << 3  | 0111011011  | 0111 01 1 011
```Shl``` R2, R1    | R2 = R2 << R1 | 0111100010  | 0111 10 0 01X
```J``` 6           | PC = 6        | 1000101110  | 1000 XX 1 110
```St``` R1, 6      | M[6] = R1     | 1001011110  | 1001 01 1 110
```St``` R2, 4      | M[4] = R2     | 1001101100  | 1001 10 1 100
```Ld``` R2, 3      | R2 = M[3]     | 1010101011  | 1010 10 1 011
```Ld``` R1, 1      | R1 = M[1]     | 1010011001  | 1010 01 1 001

---

