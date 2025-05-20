# The C runtime

The C runtime has many implementations: Both bulky and lean implementations.  

lean examples: crt0, gcrt0.  
Bulky examples: crt1, Scrt1  

We'll focus on [crt0](https://en.wikipedia.org/wiki/Crt0) (insert even more sad noises)  
Just the Init part.  

## More levels  
There are also many implementations of Crt0's. Some have more functionalities than others.  
You have to choose a lib that suits your needs. 


For example look at this marvel here: [Visual Studio C runtime reference](https://learn.microsoft.com/en-us/cpp/c-runtime-library/c-run-time-library-reference)  
A lot of [functions](https://learn.microsoft.com/en-us/cpp/c-runtime-library/run-time-routines-by-category?view=msvc-170), so so many.


forget all the other step-ups, let's just focus on crt0.   


# So the Crt0

crt0 (also known as c0) is a set of execution startup routines linked into a C program that performs any initialization work required before calling the program's main function. After the main function completes the control returns to crt0, which calls the library function exit(0) to terminate the process.  

Crt0 is automatically included by the linker into every executable file it builds. (you can disable this with a custom linker script)



