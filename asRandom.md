# Address space randomization

## countermeasures to the bufferoverflow
- placing canary next to return address (check if it got changed)
- double checking the return address
- reordering local variables and arguments, even put the canary on the stack
- copies function pointers to an area preceding local variable buffers

## W⊕X Pages and Return-to-libc
- W⊕X is based on the observation that most of
the exploits so far inject malicious code into a process’s address space and then circumvent program
control to execute the injected code.
- Under W⊕X, pages in the heap, stack, and other memory
segments are marked either writable (W) or executable (X), but not both.
- Traditionally, attackers have chosen to call functions in the standard C-language library, libc,
which is an attractive target because it is loaded into every Unix program and encapsulates the
system-call API by which programs access such kernel services as forking child processes and communicating over network sockets. This class of attacks is therefore known as “return-to-libc.”
- the attacker can't just execute the code, they will need to return to another function to execute.

## ASRandom:
- above attack will need to know virtual address of libc to write into the ret or pointer
- randomize the address will help defend
- one implementation randomizes the base address of the stack, heap, and code segments and adds random padding to stack frame and malloc() function calls
- solution to the fact there is only limited space: periodically relink and recompile executables and libraries

## Case of PaX ASLR design:
- For the purposes of ASLR, a process’s user address space consists of three areas, called the executable, mapped, and stack areas
- ASLR randomizes 3 areas individually and add offset that's randomly chosen when process is created

- **attack side** 
- because PaX ASLR randomizes only the base addresses of the three memory areas, once any of the three
delta variables is leaked, an attacker can fix the addresses of any memory location within the
area controlled by the variable
- In particular, we are interested in the delta mmap variable that determines the randomized offset of segments allocated by mmap().
- Second, in PaX each offset variable is fixed throughout a process’s lifetime, including any processes that fork() from a parent process
- delta_map: map data offset

## Return to Libc attack: (example of apache server)
- step of attacking: gettting delta map
    1. get the value of delta map by keep overflowing stack buffer with guesses to return into the libc
    2. For each value of delta mmap, compute the guess for the randomized virtual address of usleep() from its offset.
    3.  Create the attack buffer (described later) and send it to the Apache web server.
    4. If the connection closes immediately, continue with the next value of delta mmap. If the connection hangs for 16 seconds, then the current guess for delta mmap is correct.
        - By the difference of the behaviors we can get the delta map value.
    5. compute address to other functions because we have delta map already


## How to improve the asRandom
- 32 bits address are ez to defeat, 64 is not
- randomize the AS layout more frequent (periodic random doesn't work that well after the first shuffle)
    - case 1: ASRandom is fixed during the attack, easy to be defeated and calculated the result
    - case 2: ASRandom changed each crashes or runs. better security
- increasing the frequency of address-space re-randomization is at best equivalent to increasing the entropy of the address space by only 1 bit
- randomize the offset even finer
    - random it during compile
    - random it during runtime
        - obviously more than 16 bits
        - reordering functions and use indirect jump instead (which is more costly)
        - reordering:
            - if we want to randomize the ordering, we need to take care of page alignment of functions, which is one of the advantage of the libraries
            - this will cause us to need to use 
            - this will cause us to need to use something better than GOT, which haven't been found yet
    - so far best thing we can do is 16 - 20 bits on 32 bits system
    - monitor and error catching
        - shut down process if something is wrong
        - alert admin
        - too much holes

