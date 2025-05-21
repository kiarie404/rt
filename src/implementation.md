# Implementation

- **Inline Assembly** { `asm!` and `naked_asm!`} : , the assembly code is emitted in a function scope and integrated into the compiler-generated assembly code of a function. The naked attribute prevents the compiler from emitting a function prologue and epilogue for the attributed function.

- Module-level inline assembly. {`global_asm`} : the assembly code is emitted in a global scope, outside a function. This can be used to hand-write entire functions using assembly code, and generally provides much more freedom to use arbitrary registers and assembler directives.

- using separate files