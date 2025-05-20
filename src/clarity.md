# Clarity



## What is the difference between a loader and a runtime? (in an environment with a kernel)
Both set up parts of the execution environment, but they operate at different layers and handle distinct responsibilities. 


The loader is at a lower level. It comes before the Crt0. 
(typically*: Before your program starts (kernel hands control to the loader first).)  

- It fetches a program from secondary memory and loads it into the primary memory. (both the executable and the needed dynamic libraries)  
- It relocates addresses accordingly. (eg: the included libraries get addresses that are in prose with the main executable)
- It resolves dynamic symbols (e.g., functions in libc).  
- Sets up the process memory layout (stack, heap, segments). (depends on the kernel involved)


The runtime on the other hand 
- Initializes the C environment after the loader finishes
 - makes sure `libc` functions can be called
 - takes care of duplicating thread variables
 - Sets up `argc/argv/envp` for main(): copies them from the stack and passes them to main() before calling main().  
 - Initializes .bss (zero-filled data) and .data (static variables).
 - ensures that the main() func will return control to the `exit` functions. (Calls destructors in reverse order)

loader: Comes before Crt. It loads programs into memory. 



# CRT0 vs CRT1
CRT1 has more functionalies than CRT0; zero is more of bare-metal while one is of the assumption of a unix environment.   
So CRT0 assumes `libc` is absent while crt1 assumes `libc` is present. (part of its function is to make sure functions like `printf` have a buffer to write to).  

The flow is different: 
0. **zero**: _start --> main() 
1. **one**: _start -->  __libc_start_main --> main()

crt1.o is the default startup file for C/C++ programs on modern Unix-like systems (Linux, BSD, etc.)  
They are not stages; they are completely different files.  

Three major differences in functionaliy: 
1. CRT1 Initializes thread-local storage; but crt0 does not have this feature
2. CRT1 provides for libc through Via __libc_start_main; but in crt0, you have to provide a manual 'init' if you want to support libc
3. CRT1 has a more standard exit strategy unlike crt0 where the exit strategy is manual (often an infinite loop)



## C-standard library vs C runtime
`c-lib` : : A collection of standardized functions (ISO C spec) like printf(), malloc(), strlen().  
`crt0`: The low-level code that runs before and after main().  

The `c-lib` links to `crt0`.  
They are separate files (modulated development can be achieved)

example implementation: glibc:	Implementation (Linux) + CRT  == libc.so.6  {demo this}
