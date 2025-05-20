## Examples of other runtimes

1. gcrt0: [source](https://github.com/eblot/newlib/blob/master/libgloss/rl78/gcrt0.S) : alternative to crt0 if you want to do some performance profiling (preferably with [gprof](https://ftp.gnu.org/old-gnu/Manuals/gprof-2.9.1/html_mono/gprof.html)). It Initializes profiling timers (monstartup) and writes gmon.out at exit for analysis.
2. Those that assumne Position-Independent Executables / PIE are being loaded : eg 



- rust runtime
- glibc
- musl
- msvcrt (Windows),