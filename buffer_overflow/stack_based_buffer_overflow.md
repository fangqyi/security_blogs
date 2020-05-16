## Stack Based Buffer Overflow Exploitation

#### Stack Based Buffer Overflows:

- affects any functions that copies input to memory without bounds checking
  - _printf()_
  - _sprintf()_
  - _strcat()_
  - _strcpy()_
- _memcpy()_
  - _gets()_
  - etc
- overflow: when the source data size is larger than the destination buffer size, it overflows the buffer towards higher memory address and overwrites previous data on the stack (as the stack grows towards lower address)
- safe alternative in C:
  - base C language: _fgets_ to _gets_
  - Microsoft version of C: _sprintf_s_, _strcpy_s_, _strcat_s_

#### A Pipeline for Basic Exploitation:

1. Write past array buffer ending and overwriting EIP register to crash the program.
2. Find the offset of the payload after which the EIP is overwritten.
3. Find and remove bad characters
4. Find the address of the _JMP ESP_ opcode so that program control flow can be redirected to the stack.
5. Overwrite return address at EIP with the address of _JMP ESP_.
6. Generate the payload and exploit the program.

#### Exploiting:





#### Attempts to Mitigate Exploits on Buffer Overflow:

- Stack canaries
  - The _canaries_ are the special values that the compiler places on the stack between the location of the buffer and the location of control data.
  - When a buffer overflow occurs, the canary will be corrupted first and be detected immediately.
  - The compiler extensions that use canaries: StackGuard and ProPolice
- Address Space Layout Randomization (ASLR)
- Executable Space Protection (Windows: Data Execution Prevention)
  - The mechanism prevents the execution of code that is located in the memory space assigned to the stack, heap, or other areas, making it impossible to directly call a shellcode but attackers may use advanced tricks such as return-oriented programming.