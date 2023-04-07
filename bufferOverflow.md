# Processes: (from low to high mem address)

## Text:
- The text region is fixed by the program and includes code (instructions) and read­only data
- Read-only and edit will get the segmentation violation


## Data:
- The data region contains initialized and uninitialized data
- Static var saved here
- Data-bss of the exe file
- If the expansion of the bss data or the user stack exhausts available memory, the process is blocked and is rescheduled to run again with a larger memory space
New mem is added between data and the stacks 

## Stack:
- The stack is also used to dynamically allocate the local variables used in functions, to pass parameters to the functions, and to return values from the function. 
- We used stack for procedure call to return to control the calls.
- Stack pointer point to the top  of the stacks and the bottom of the stack is the fixed address.
- CPU implements the instructions to PUSH or POP
- many compilers use a second
register, FP, for referencing both local variables and parameters because their distances from FP do not change
with PUSHes and POPs. On Intel CPUs, BP (EBP) is used for this purpose. Essentially it doesn't move with the stack changes

## Functions call:
- when function is called, Instruction pointers (IP) will push the return address onto the stack (RET)
- Then EBP will be push on the Stack and current Stack pointer will be store in there
- If there is local var then subtracts the local var from the current stack pointer for space

## Buffer Overflow:
- overwritten the stack beause of the running of the space. (high level) For example, run a function of putting a 256 bytes array into the 16 bytes array will cause it to overwrite the stack even the RET and even EBP
- RESULT: we can control the return address of the program and hijack the processes

## Shell Code:
- place the codes we want to execute in the buffer we are overflowing and overwrite the return address to the buffer, in most cases we will want a shell
- C shell code looks like below:

```c
#include stdio.h
    void main() {
        char *name[2];
        name[0] = "/bin/sh";
        name[1] = NULL;
        execve(name[0], name, NULL);
    }
```
