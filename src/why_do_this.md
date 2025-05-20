# Why do all this? What's the point? 

There is no point but learning. <sad noises>  


You might be able to :
  - tweak existing runtimes to suit your project. (if you are developing software that runs close to the hardware, this is almost inevitable)
    - remove bloat : (e.g., skip _init of libc if not needed).  
    - replacing generic libs with specialized libs (eg the handling of memory allocation of the heap in a mor determiistic way) 
    - if you are on bare-metal, you are the runtime
  
  - write your own runtime for your new environment. 
  - Security & Hacking : Writing secure code requires knowing what the runtime does behind the scenes.  
