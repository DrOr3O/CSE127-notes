# Processes: Text, Data, and stack. (from low to high mem address)

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
- Then EBP will be push on the Stack to save the frame before going into the function. 
- If there is local var then subtracts the local var from the current stack pointer for space
```
0x8000130 : pushl %ebp
0x8000131 : movl %esp,%ebp
0x8000133 : subl $0x8,%esp

```
- first line is function call, we store the main frame into the stack
- second line we store the current function frame into the ebp (so we know which frame it is now)
- Third line size can vary because it is allocating the space for the local variables. 


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
- Now we hav the shell code that can follow the assembly to execute and have those strings somewhere in the memory, the problem being we don't know where we can find it.
- Use JMP and Call to work around, sicne they use relative addressing. which means we can jump to an offset from the
current Instruction Pointer without needing to know the exact address of where in memory we want to jump to. If we place aCALL instruction right before the "/bin/sh" string, and a JMP instruction to it, the strings address will be
pushed onto the stack as the return address when CALL is executed.
- no NULL bytes in the shell code to function properly.


### Quick version of what the shell codes are accomplishing:
1. Have the null terminated string "/bin/sh" somewhere in memory. 
2. Have the address of the string "/bin/sh" somewhere in memory followed by a null long word.
3. Copy 0xb into the EAX register.
4. Copy the address of the address of the string "/bin/sh" into the EBX register. 
5. Copy the address of the string "/bin/sh" into the ECX register.
6. Copy the address of the null long word into the EDX register. 
7. Execute the int $0x80 instruction. 
8. Copy 0x1 into the EAX register.
9. Copy 0x0 into the EBX register. 
10. Execute the int $0x80 instruction. 

## Other notes for how to effectively know the location of the buffer in the memories:
- For every program we are trying to execute its stack is likely start at the same address. 
- In order to increase the chances, we can use NOP null instruction to pad our shellcode

## Small BufferOverflow
- Either buffer is too small for shell code or NOP is too small for guesses.
- Diff approach, only work when we can access the enviornmental variable
- Store the shellcode into the Enviornment variable then overflow buffer with address of this variable 
- The environment variables are stored in the top of the stack
when the program is started, any modification by setenv() are then allocated elsewhere

## Finding Overflow
- C have many copy and paste function without bound checking:  strcat(), strcpy(), sprintf(), and vsprintf()


# Questions for TA:
- What is this $EGG about and how exactly does the function works.
