## Buffer-related

#### Buffer

Buffer is a memory space which is used to store data temporarily (mainly located in DRAM) - to counter the difference between the rate at which data is received and the rate at which it can be processed ( or rates are variable e.g. online video streaming).

It often adjusts timing by implementing a queue/FIFO algorithm in memory, simultaneously writing data into the queue at one rate and reading it out at another rate.

#### Memory Layout (in C)

![Memory-Layout](https://media.geeksforgeeks.org/wp-content/uploads/memoryLayoutC.jpg)

<div align="center">credits:[GreeksforGreeks](https://www.geeksforgeeks.org/memory-layout-of-c-program/) <div>

##### Text/code segment:

- contains executable instructions (in an object file/ in memory)
- placed below the heap or stack to prevent being overwritten by the overflows
- read-only to compilers, text editors, & the shells (which frequently runs the program)

##### (Initialized) Data Segment:

- a portion of virtual address space of a program, which contains _global & static_ variables initialized by the program

- can be read-write for vars that need to be changed at runtime

  ```c
  char s[] = "hellow world"; //outside the main 
  ```

- can be read-only

  ```c
  const char* string = "hello world";
  ```

##### Uninitialized Data "bss" Segment:

- "bss" for _block started by symbol_

- initialized by the kernel to arithmetic 0 before the execution

- starts at the end of the data segment

- examples:

  ``` c
  int global; //outside main
  static int global_static;
  ```

##### Stack:

- traditionally adjoined the heap & grew the opposite direction
- when the stack pointer met the heap pointer, free memory exhausted 
- (with virtual memory technique & modern large address spaces, they could be placed almost anywhere)
- LIFO
- located in the higher parts of memory, grows toward address zero (PC x86); some grow in the opposite direction
- a "stack pointer" register tracks the top of the stack & adjusted each time a value is pushed onto the stack
- "stack frame": a set of values pushed for one function call & consists at minimum of a return address 
- each time a function is called, 
  - the address of where to return to & info about the caller's env (some of the machine registers) are saved on the stack
  - then allocates room on the stack for its automatic & temp vars

##### Heap:

- the segment where dynamic memory allocation usually takes place
- begins at the end of the BSS segment 
- grows to larger address
- manage by malloc, realloc, and free
- brk, sbrk system calls to adjust its size



#### Buffer overflow

It can be caused by poor coding practices on unchecked boundary. [Simple code example](https://blog.rapid7.com/2019/02/19/stack-based-buffer-overflow-attacks-what-you-need-to-know/)

It could be used to overwrite certain targeted data.

Links: 

[Data Buffer - Wikipedia](https://en.wikipedia.org/wiki/Data_buffer)

[Memory Layout of C Programs](https://www.geeksforgeeks.org/memory-layout-of-c-program/)



