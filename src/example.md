# Example of global feature

```rust

#![feature(global_asm)]

// Simple RISC-V assembly function that adds two numbers
global_asm!(r#"
    .section .text
    .globl riscv_add
    .type riscv_add, @function
riscv_add:
    add a0, a0, a1  # RISC-V calling convention uses a0-a1 for args/return
    ret
"#);

// Declare the external function
extern "C" {
    fn riscv_add(a: i32, b: i32) -> i32;
}

fn main() {
    let result = unsafe { riscv_add(10, 20) };
    println!("Result from RISC-V assembly: {}", result);
}

```  


Stick to ABI conventions:

Calling Convention:
- Arguments: a0-a7
- Return value: a0
- Caller-saved: t0-t6, a0-a7
- Callee-saved: s0-s11
