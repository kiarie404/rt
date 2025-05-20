## Inline Asm


Why you would want to mix rust with asm: 
- you want to use/exploit a specific asm insruction
- you want to manipulate the machine at a lower level eg when writing runtime code or a loader  


# Method 1: Inline Assembly
You can embed assembly using the `asm!` macro.  
It is unsafe because the compiler cannot check whatever magic you are doing with your assembly. It cannot guarantee safety.  

(undone: example)

## (maybe limitations)/warnings
1. We cannot use value of type `&mut T` or `&T` for inline assembly; only integers, floats, SIMD vectors, pointers and function pointers can be used as arguments for inline assembly eg this would fail to compile
```rust
fn increment(variable: &mut u64){
    unsafe asm!( " mov {}, 1 ", out(reg) variable);
}
```

2. It is assumed that `out` register get written to in the asm instructions, so the variable representing the out must be mutable.  For example, this will not compile : 
```rust

// this will NOT compile...
fn increment (var: u64){
    unsafe {    asm!("add { }, 1", out(reg)var);    }
}

// However, this will compile... however, you will be modifying a copy 
fn increment (mut var: u64){
    unsafe {    asm!("add { }, 1", out(reg)var);    }
}

// Now this will modify the initial variable (but it will be pure garbage)
fn increment (var: &mut u64){
    unsafe {    asm!("add {}, 1", out(reg)(*var)); }
}
```

3. If you mess up the reg-type, you will not get a compilation error. But your code will not function as expected. Here are 2 examples:
   1. example 1: no changes happen when the assembly code modifies an in-register (no compiler errors detected)
   ```rust
        let mut x: u64 = 0;
        
        // we want to make the value of x be 1 ie. x = 1
        unsafe { asm!( "mov {}, 1", in(reg)x); } // instead of out(reg)x, we put in(reg)x. This is intentionally wrong

        assert_eq!(1, x); // // this will panic because x will still be 0. in-registers do not get modified...the compiler did not warn us

   ```
   2. example 2: reading from an out register gets you garbage. Again, the compiler did not give a hoot.  
   ```rust
        let mut x: u64 = 2;
        let mut y: u64 = 0;
        unsafe {  asm!( "add {0}, {1}", out(reg)y, out(reg)x); }

        assert_eq!( y, 2); // this will panic because the addition operation read from garbage
   ```
4. The above consequences are scary, garbage computation goes undetected. To be safe, you can choose to make all operands `inout`. But that may make your code harder to debug since it will not be explicit when a variable gets modified...moreover, `outs` and `inouts` are memory-expensive and they affect performance. (this has been covered in the register allocation section)

5. Surprises in how the assembly instructions work, especially in CISC architectures like x86. The little implicit assembly details that ruin your code.  
   Based on the above warnings, it is obvious that if you mess up the `reg-type` you either get garbage or incorrect results. What's worse is that some certain assembly instrutions do more than what you might expect them to do. For example... here is an instruction that just decides to read from an output. Quite surprising!
   ```rust
        let mut x:u64 = 1;
        unsafe {  asm!("add {}, 1", out(reg)x); } // our intuition says that x should be an out-register... right?
                                                  // but this add instruction implicitly uses the 1st operand as an both input and output:
                                                  // this is because the add instruction is made up of 2 sub-instructions

        assert_eq!(x, 2); // this line panics because x contains garbage. This is because the innocent `add` instruction treats the first operand
                          // ... as both an input and an output (this is not obvious unless you've seen the implementation of the add instruction)

        // Here is a fix....
        let mut x:u64 = 1;

        unsafe {  asm!("add {}, 1", inout(reg)x); } // now this works as expected
        assert_eq!(x, 2);
   ```
6. From the above caveats,you realize that you can just decide to use `inout` everywhere. This is a bad decision because `out` and `inout` are memory expensive. We will learn about this cost in the **register allocation** section.  

## The `inout` register type  
A type that may get read-from and written-to in the assembly code.  
So this type MUST be `mut` or you will get a compilation error.  
For example:  
```rust
    let mut x = 1;
    unsafe {  asm!( "add {0}, {0}", inout(reg)x); }

    assert_eq!(2, x);
```

It is also possible to specify different variables for the input and output parts of an inout operand. But this is somehow unreadable...less explicit.  
```rust
    let x:u64 = 5;
    let mut y:u64 = 0;

    unsafe { asm!(" add {0}, 3", inout(reg)x => y); }
    assert_eq!(y, 8);
```


## Register Allocation
When the compiler allocates operands to the different registers it has to be careful in order to avoid mix-up of data.  
This is the hard subject of register allocation. he he he ... so I won't talk about it. I do not have the details. Go find a book that does. Bye!  


## How Register allocation happens
(undone)

But there is one thing I know. The processor has very few registers, so the compiler breaks down the code into small basic units of execution that can each sufficiently use those few registers. So from there, it does some sort of context switching...but between those basic units of computation. Those basic units share the limited registers this way.  

Another method of making sure the registers are shared 

### Late output operands    
(undone: explanation and example in riscv)

### Explicit register operands
Some assembly code requires you to use specific named registers eg control status registers.  
This part is architecture specific. We will use riscv because ... well ... because... because RISCV is the chosen one.  
The ISA above all ISAs.(insert delulu laughter)  

(undone in ricv)

### Clobbered Registers
Ideally, the inline assembly should only modify the state of `out`, `inout`,`lateout` and `inlateout` registers that you declare. And in case you use explicit registers eg the control status registers, then it should also just affect those registers.  
Point is, the assembly code is expected to only affect the registers that you pass to it for moification.  

But in many cases, the inline assembly code uses more registers eg scratch registers to store temporary values in calculations. This registers that get caught in the cross-fire are referred to as **`clobbered registers`**. We need to tell the compiler to restore the state of these `clobbered` registers just to be on the safe side. 

Think of this as the good old caller-saved and callee-saved registers. This safety comes at a cost though, sigh.   

(undone: riscv example messing up the clobbers vs restoring the clobberss)  
(undone: you can view the way clobber operans have been used by inspecting the `clobber_abi` argument to the `asm!` macro)

By default, `asm!` assumes that any register not specified as an output will have its contents preserved by the assembly code. The `clobber_abi` argument to `asm!` tells the compiler to automatically insert the necessary clobber operands according to the given calling convention ABI: 

(show example)

### Register template modifiers
(undone: riscv explanation and example)

### Memory Address Operands
Sometimes assembly instructions require operands passed via memory addresses/memory locations.  
For this, you just stick to the syntax of your assembly language. No rust formatting gets involved. 

(undone: riscv example)

### Labels 
Discuss labels in compilation process.  
Any reuse of a named label, local or otherwise, can result in an assembler or linker error or may cause other strange behavior.  
(undone: show failing and passing code for each of the situations below)  

Advice; AVOID LABELS IN INLINE ASM

### Why do people say inline assembly is safer than compiling Rust code with seperate pure asm files?
- clobbered registers automation via `clobber_abi` arguments

### Why it may not be safer
- no errors when reg-type is mismathced
- no compiler errors or warnings when labels clash
- if no clobber-abi is used, bugs are impossible to find

### Options
These are hints that you pass together with your asm. The Hints help the compiler to optimize your assembly.  
Here are the current hints:
1. **pure**: indicates that the asm block has no observable effects and that the output depends only on the explicit input. Think of it like a constant function. This means that the compiler can re-use the results if the same inputs are repeated. The compiler can even replace the entire asm code with a constant value if it's semantically correct. 
2. **nomem**: informs the compiler that the asm code does not read/write to memory. Just register-register or register-immediate action. This helps the compiler in remoing memory synchronization guards around the code. It also makes it easy for the compiler to hint the hhardware below of inconsequential ordering.
3. **nostack**:  means that the asm code does not push any data onto the stack. This might help the compiler to remove the extra stack management around the code


### Performance
- Cut on the expence of 
  - clobbered registers
  - out registers
- Use options in our inline assembly to hint to the compiler what your code is doing






## Direction specifications
- in: a variable that will just be read-from
- out: variable will be written into
- inout: A type that may get read-from AND written-to in the assembly code. 
- lateout
- inlateout

## Register specification
- register_class : 
  - reg : general register, compiler gets to pick which register is free
  - 
- \reg_name\
**Alternatives**  
For stable Rust, consider using:
- The riscv crate for CSR access
- The riscv-rt crate for startup code
- External assembly files for larger blocks