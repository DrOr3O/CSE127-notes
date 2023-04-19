## Buffer overflow vs Format String Vulnerabilities
- number of exploits *vs* a few thousand a few dozen
- considered as security threat *vs* programming bug
- techniques evolved and advanced *vs* basic techniques
- visibility sometimes very difficult to spot *vs* easy to find


## Format function
- A format function is a special kind of ANSI C function, that takes a variable
number of arguments, from which one is the so called format string.
- conversion to make human more readible 
- the function evaluates the format string, it accesses the extra parameters
given to the function
- printf always take other argument beside the string

### the printf family:
fprintf — prints to a FILE stream
- printf — prints to the ‘stdout’ stream
- sprintf — prints into a string
- snprintf — prints into a string with length checking
- vfprintf — print to a FILE stream from a va_arg structure
- vprintf — prints to ‘stdout’ from a va_arg structure
- vsprintf — prints to a string from a va_arg structure
- vsnprintf — prints to a string with length checking from a va_arg structure
Relatives:
- setproctitle — set argv[]
- syslog — output to the syslog facility
- others like err*, verr*, warn*, vwarn*

### the functionality of the format functions:
- used to convert simple C datatypes to a string representation
- allow to specify the format of the representation
- process the resulting string (output to stderr, stdout, syslog, ...)

Usage:
- the format string controls the behaviour of the function
- it specifies the type of parameters that should be printed
- parameters are saved on the stack (pushed)
- saved either directly (by value), or indirectly (by reference)

Calling: Has to know how many parameter to push on stack since it will do stack correction when function returns

## Stack and role of the formatting:
- see images

## format string vulnerabilities:
- usually occurs when format string parameter is usersupplied or indirectly sent into it
- behaviour of the format function can be control through supplying the format string 

## the attack and crashes
- in all UNIX, illegal pointer access are caught by kernel and process are sent a SIGSEGV signal and terminated 
```
printf ("%s%s%s%s%s%s%s%s%s%s%s%s");
```
- this will crashes beacuse we are keep asking to print the mem address on the stack

- to view stack memories we can do:
```
printf ("%08x.%08x.%08x.%08x.%08x\n");
```
- this shows the 5 parameters on the stack, from bottom towards the top of the stack

- we can supply address to view any mem (by reference)
- we have to supply formate parameter which supply address as stack parameter and the actual address
- format function internally has a pointer point to stack location of current format parameter
- we need to get this pointer to point at the mem space we control 
- In a two-stage process, first a saved instruction pointer is overwritten and
then the program executes a legitimate instruction that transfers control to
the attacker-supplied address.

# exploit
- the good ol way of bufferoverflow through the format string parameter expand the words limitation
- or exploit certain format string saving the data with a pointer to certain address
- for example, something like a number we can easily control by changing the parameters (1 byte)
- for address we need to control 4 bytes completely 
```
unsigned char canary[5];
unsigned char foo[4];
memset (foo, ’\x00’, sizeof (foo));
/* 0 * before */ strcpy (canary, "AAAA");
/* 1 */ printf ("%16u%n", 7350, (int *) &foo[0]);
/* 2 */ printf ("%32u%n", 7350, (int *) &foo[1]);
/* 3 */ printf ("%64u%n", 7350, (int *) &foo[2]);
/* 4 */ printf ("%128u%n", 7350, (int *) &foo[3]);
/* 5 * after */ printf ("%02x%02x%02x%02x\n", foo[0], foo[1],
foo[2], foo[3]);
printf ("canary: %02x%02x%02x%02x\n", canary[0],
canary[1], canary[2], canary[3]);
```
- overwrite four times the least significant byte of an integer we point to. By
increasing the pointer each time, the least significant byte moves through
the memory we want to write to, and allows us to store completely arbitrary
data.

- <stackpop><dummy-addr-pair * 4><write-code>
- stack sequence to increase stack pointer
- stackpop: The sequence of stack popping parameters that increase the stack
pointer. Once the stackpop has been processed, the format function
internal stack pointer points to the beginning of the dummy-addr-pair
strings.
- dummy-addr-pair: Four pairs of dummy integer values and addresses to
write to. The addresses are increasing by one with each pair, the
dummy integer value can be anything that does not contain NUL bytes
- write-code: The part of the format string that actually does the writing
to the memory, by using ‘%nu%n’ pairs, where n is greater than 10.
The first part is used to increase or overflow the least significant byte
of the format function internal bytes-written counter, and the ‘%n’
is used to write this counter to the addresses that are within the
dummy-addr-pair part of the string.

## Varoation of exploitation
rest of the text are talking about variation of methods.

