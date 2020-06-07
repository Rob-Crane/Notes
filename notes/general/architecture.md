# Architecture

## Booting
1. At reset, CPU loads first instruction from hard-coded address, the "reset vector".  The motherboard directs this special memory address to wherever it stores its firmware (instead of the memory bus) to load BIOS (old) or UEFI (new).
1. BIOS/UEFI first executes Power On Self Test (POST) and then searches through (possibly configurable) list of storage devices.  At each storage device, BIOS/UEFI loads a fixed size portion at the beginning of storage device, the Master Boot Record (MBR, old) or GUID Partion Table (GPT, new).
1. The instructions contained in this record describe the partioning of the desk but also flag if the device is bootable.  If so, it will also intructions for loading the operating system kernel.  The instructions can either directly load the kernel (if it's relatively small) or it can contain instructions to load a more complicated, larger second-stage bootloader like GRUB.

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
### Calling Conventions
* From 10,000 feet, to call a function, a "caller" needs to know where to put arguments and where to access any return value.  The "callee" needs to know where to access arguments and where to put values to return.  The function being called may be a system call.  These expectations are standardized as "calling conventions".
* Calling conventions also include requirements to preserve register values, type representation, and name mangling (convention for symbol names required by linker) together form the Application Binary Interface (ABI) specified by operating systems.
* Stack conventionally grows "downwards" so decrementing SP "grows" the stack.
  * **Example:** `pushq %rbp` Decrements RSP (stack pointer) by 8 and writes contents of RBP to location pointed to by RSP.
* Procedure call begins with `call [label]` which, after incrementing instruction pointer (IP), pushes it on the stack.  IP then populated with instruction address of label.  When procedure call ends, stack is returned to this condition.  `return` inverts this process by popping instruction address off stack and jumping to it.
* Some x86-64 ABI rules ([source](https://cs61.seas.harvard.edu/site/2018/Asm1/)):
  * Caller stores first six arguments in registers  %rdi, %rsi, %rdx, %rcx, %r8, and %r9 if possible.  Otherwise on stack.
  * Return value is stored in %rax if it fits.  Otherwise, caller reserves space on stack and passes address as first argument to callee.
* The base pointer RBP (or frame pointer) provides a fixed reference to the stack frame of the currently executing function.  This is populated at the beginning of a procedure as follows:
```asm
	pushq	%rbp       ; Push caller's base pointer onto stack to preserve it.
                  ; Stack pointer now points at preserved RBP
                  
	movq	%rsp, %rbp  ; Populate RBP with current stack pointer.
                  ; Since (%rbp) is the caller's frame pointer, can trace
                  ; up call stack by following chain of frame pointers
                  ; saved on stack.
```
([illustration](https://cs61.seas.harvard.edu/site/img/stack-2018-03.png))

## Interrupts (x86)
Normal program control flow consists of jumps and procedure calls.  Interrupts provide an additional mechanism and can be triggered within the program executing with the `int` opcode or externally on an IRQ line (typically from a Programmable Interrupt Controller (PIC) which contains additional logic for multiplexing/queueing interrupts).

When an interrupt is executed it is executed with a number which identifies the interrupt.  In protected mode, the interrupt indexes into an Interrupt Descriptor Table (IDT) - similar to Global Descriptor Table for segmented memory management - which contains address of handler procedure.  In real mode at boot time, the IDT doesn't exist (since it's created by the OS kernel).  Interrupts instead index into a simpler Interrupt Vector Table (IVT) loaded by BIOS.  These interrupt handlers are defined by the BIOS manufacturer and can be used like "system calls" to BIOS.  These BIOS interrupts do not conform with Intel's specification for interrupt numbering ([source](https://stackoverflow.com/questions/62122049/how-are-bios-interrupts-deconflicted-with-reserved-hardware-interrupts)).

Actually using interrupts is a way of making system calls from normal programs but more recent processors have opcodes ([SYSENTER/SYSEXIT](https://cs.stackexchange.com/questions/38141/why-system-calls-via-interrupts-are-slow-and-thus-we-have-sysenter-sysexit-instr)) which make system calls more efficiently with lower overheads than interrupt handling.

Interrupts are used asynchronously by external hardware to signal the processor to handle an event (like a keystroke) and synchronously for things like system calls and error signalling. (divide by zero, etc.)

### Context Switching
A special class of interrupt is a *timer interrupt* which is used to support context switching between processes.  It originates from a hardware timer - historically a Programmable Interval Timer (PIT), more recently, High Precision Event Timer (HPET).  A timer interrupt serves only as a signal to pause the currently executing process.  The interrupt handler executes the OS's process scheduler which will decide the next process to execute.  If a different process is selected, the OS will execute a context switch.

The [Task State Segment](https://en.wikipedia.org/wiki/Task_state_segment) (TSS) is a data structure which holds information about the currently executing thread.  The task register (TR) indexes into the Global Descriptor Table (GDT) for the segment descriptor which points to the current task's TSS.  Among other things, the TSS holds a pointer to the thread's "kernel stack"  which resides in kernel space for privileged routines like system calls and fault handlers.

In the context of a context switch, the thread's state (registers, stack pointer, etc.) are pushed onto its kernel stack.  The OS then moves to the new thread's TSS and restores the CPU with state from its stack.  When the return-from-trap is executed, the new thread is now running.
