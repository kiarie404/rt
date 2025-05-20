# Explanations


The best way to understand something is to inspect its algorithm :  
(We will assume that the loader is not available and that we are dealing with an elf-file.)

The elf file has the following sections:  
- **.text**: Executable code 
- **.data**: Initialized global variables
- **.rodata**: Read-only data (const, strings) 
- **.bss**:	Uninitialized globals `(int y;)	y: .space 4`
- **.stack**: Runtime stack (local vars, return addresses)  
- **.heap**: Dynamic memory 


## The steps (CRT0)

### 1. Set Up the Stack Pointer

ASLR
Security 

Fancy word for : "Tell the CPU where the stack memory begins, so functions can use it for local variables, return addresses, and arguments." 
If you have a kernel: (CRT1)
 - The loader typically sets up the initial stack before your program starts.
 - It places argc, argv, and environment variables at the top of the stack and then crt1 just reads sp from where the loader left it.

If you’re on bare-metal (no OS) you must set sp manually in crt0. (you do the loaders work)
```riscv
la sp, _stack_top  # Load address of stack top (defined in linker script)
```  


### 2. Initialize Data Sections

1. Copy **.data section** (initialized variables) from ROM to RAM.
2. Zero out **.bss section** (uninitialized global variables). 

```riscv
# Copy .data
la a0, _data_lma  # Load ROM address of .data (Load Memory Address)
la a1, _data_vma  # Load RAM address of .data (Virtual Memory Address)
la a2, _edata      # End of .data
bge a1, a2, 1f     # Skip if .data is empty
0:
lw t0, (a0)        # Copy word from ROM
sw t0, (a1)        # Store word to RAM
addi a0, a0, 4     # Increment pointers
addi a1, a1, 4
blt a1, a2, 0b     # Loop until done
1:

# Zero .bss
la a0, _bss_start  # Start of .bss
la a1, _bss_end    # End of .bss
bge a0, a1, 1f     # Skip if .bss is empty
0:
sw zero, (a0)      # Store zero
addi a0, a0, 4
blt a0, a1, 0b     # Loop until done
1:
```  

### 3. Set Up argc/argv (If Applicable)  
For OS environments, pass command-line arguments to main(). 
Bare-Metal RISC-V: Often omitted (hardcode argc=0, argv=NULL). 

But in our case, we will assume that main takes in nothing.  



### 4. Handle main_exit, interrupts and exceptions

