# Architecture

## x86 Assembly

* `-S` option to clang/gcc causes output to be assembly (stopping after compilation and before assembly).  Can pass assembly file as input to run remaining stages of pipeline (assembly, linking).
* `objdump -S` will diassemble object code.
### Segmentation
* Since 8086 used used only 16-bit words, it used a memory segmentation scheme to address more than 2<sup>16</sup> locations.
  * **Example:** Location of the stack consisted of 2<sup>4</sup> * `SS(stack segment)` + `SP`.
* 80286 introduced an additional level of indirection in order to address more memory.  Segment registers index a "descriptor table" (in memory pointed to by descriptor table register) which contains the base address to which the relevant offset is added.  There is a single global descriptor table and local descriptor tables per process.
  * **Example:** Get location on a process' stack:
      1. Process' local descriptor table is at address in `LDTR` register.
      2. Find index into table in `SS`.  Get address base from table entry.
      3. Add `SP` to address base.
* x86-64 does not need to use segmentation scheme and zeros out segment registers.
### Registers
#### Aliasing
For historical/compatibility reasons, registers have aliases that refer to lower bits of the same register.  For example, the original 16 bit 8086 `AX` register could be addressed as two 8-bit registers `AH` and `AL`.  This became the lower bits of the 32-bit 80386 (i386) `EAX` register - which in turn became the lower bits of the 64-bit x86-64 `RAX` register.
#### Word Size
Since 8086 had a 16-bit word, "doubleword" refers to 32-bit width and "quadword" refers to 64-bit width.
### Intel/AT&T Syntax
Line of assembly structured `<op> A, B` either means:
  * AT&T Syntax: `A <op> B --> B`  **Example:** `movl $1, %eax` (move long 1 into register EAX)
  * Intel Syntax: `B <op> A --> A`  **Example:** `mov eax, 1`
### Opcode Suffixes
Letter appended to opcode can be:
  * **Data type**: `b, w, d, q` (byte, word, doubleword, quadword) or `s, d, t` (single, double, extended precision) for floating point.  If data type suffix missing, can be inferred from operand width.
  * **Sign/Zero Extension:** `s,z` for extending mismatched data types.
  * `Conditional`: Execute operation depending on contents of `RFLAGS` register bits set by previous comparison operation.
  ```asm
cmp eax, edx
ja somewhere ; "jump above" (unsigned comparison)
             ; will go "somewhere" if eax >u edx
             ; where >u is "unsigned greater than"
             ; from RFLAGS CF=0 and ZF=0

cmp eax, edx
jg somewhere ; "jump greater" (signed comparison)
             ; will go "somewhere" if eax >s edx
             ; where >s is "signed greater than"
             ; from RFLAGS SF = 0 ZF = 0
  ```
### Operand Addressing
* **Immediate ($):** Use a contant: `movq $175, %rdi` (move quad word 175 into register RDI)
* **Register (%):** Use a value in the specified register: `movq %rcx %rdi` (move quad word contained in RCX into RDI)
* **Direct:** Use a value in specified memory location `movq 0x172 %rdi` (move quad words at 0x172 into RDI)

Indirect:
* **Register Indirect:** Use value in memory address contained in specified location `movq (%rax), %rdi` (move contents at address in RAX to RDI)
* **Register Indexed:**: Adds a constant offset to address in register `movq 172(%rax), %rdi` (move contents at (address in RAX + 172) to RDI)
* **Base Indexed Scale Displacement:** Add a constant offset and a scaled index to get memory address `movq 172(%rdi, %rdx, 8) %rax` (Move contents at (address in RDI + contents of RDX * 8 + 172) to RAX)
## Common Idioms
* `xor %rax %rax`: Zero out contents of RAX
* `test` performs bitwise AND of operands, discards result, but alters RFLAGS
```asm
; Conditional Jump
test $rcx, $rcx   ; sets ZF to 1 if RCX == 0
je 0x804f430      ; jump if ZF == 1

; Jump if register value < 0
test $rcx, $rcx  ; SF (most significant bit) flag set to 0 if RCX < 0
js error ; jump if SF == 1
```
* `data16 data16 ...` No-op padding of instructions for cache line alignment.
### Floating Point/Simultaneous Instruction Multiple Data (SIMD) Instruction Sets
* Original 8086 did not provide support for floating point operations but instruction set could be augmented with functionality from dedicated x87 floating point processors.  These instructions were later integrated into subsequent x86 versions.
* MMX, SSE, AVX beginning in the late 90s were extensions which provided vectorization to execute a single instruction in parallel on multiple inputs (SIMD).  Compilers prefer SIMD instruction set versions of floating point operations over legacy x87 versions since they're easier to optimize for and more uniform with x86 opcodes.
  * SIMD opcode suffixes describe type and whether operation is scalar or vector.
    * `mulsd %xmm0, %xmm1`: Scalar multiply doubles in XMM0 and XMM1
    * `vaddpd %ymm0, %ymm1, %ymm2`: Vector add "packed" doubles in YMM0 and YMM1, store in YMM2.  "V" prefix denotes AVX instructions.
  * YMM registers are 256 bit.  XMM are 128 bit and alias lower order bits of YMM registers.
  * SIMD instruction operands can also be memory addresses but some instructions require certain alignment.
