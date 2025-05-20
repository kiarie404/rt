# Runtimes

What is a C runtime? What is a Rust runtime? What is a runtime?  


## 100 Meanings
Well, the word runtime is a dense word, with multiple meanings that vary with context. Here are some of the definitions :  

### Meaning 1 :  
Runtime refers to the duration of time consumed when a program was executing.  
For example, if you played a videogame for 12 hours, you could say that that videogame had a 12-hour runtime.   

### Meaning 2 :  
Runtime refers to a piece of software that continuously tends to the needs of another running program.  
For example :  a garbage collector is part of a program's runtime.   


### Meaning 3 : (misnomer?)
Programs usually don't start getting executed just like that. There has to be code that makes the CPU to point to and fetch the right lines of code, make sure that there is enough stack space, make sure that some CPU registers have the expected default values...  

Point is, there is some code that gets executed before the actual program ie. Init code AND control code.  

In this context, Runtime means init-code. Runtime means control code.  
Some parts of the runtime code get executed once while other parts get executed continuously & dynamically.  

For example, control functions like *overlay control*,*stack overflow detection* and *Exception Handling* get executed continuously & dynamically. Functions like program-stack initilization get executed only once at the start of the program.  

Some compilers can be set to statically insert such control code in every place they are needed at compile time. Other compilers allow you to reference them dynamically.  

If you are short on space, you can go the dynamic path.  
If you are not short on space and you require performance, it's better to go the static path.  


A runtime is a layer of code that executes before and after your main() function. -- (it also runs while the main program is running)  
So a runtime might handle all or at least one of the following:
- Startup service
- Shutdown/Clean up service
- mid-run services



