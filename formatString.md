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

## What is a format String?
