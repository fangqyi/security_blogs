## Stack-based Buffer Overflow

#### Stack

- LIFO
- consists of frames which are pushed when a function is called and popped when a function is finished
- faster than heap due to its simplicity
- memory would be reclaimed when the creating function exits
- declare more memory than available in CPU can cause crashes - functions that uses _alloca_ (allocate memory that is automatically freed _void alloca(size_t size)_) are generally prevented from being inlined (integrate the function's code into the code for its callers)
- stack frame: a frame of data that gets pushed onto the stack
  - call stack: a stack frame would be a function call & its argument data (return address is pushed first, then the args, then local variables)
- in intel_x86 architecture: max data size would be a WORD (4 bytes long for each push or pop)
- interfaces: compiler typically translates commands to inlined instructions manipulating the stack pointer (similar to the handling of variable-length arrays - determined at run time)
  - Unix-like: _alloca_, _malloca_, _freea_
  - C: _malloc_, _realloc_, _calloc_, _free_

![img](https://miro.medium.com/max/1000/1*1Or1lGJksfMPGUqHOZXVUw.png)

<div align="center">credits:[Wikipedia](https://en.wikipedia.org/wiki/Stack-based_memory_allocation) </div>

#### Registers

- EAX: accumulator (general purpose: can store data & address) - used for performing calculations, and used to store return values from function calls
  - for basic operations such as add, subtract, compare 
- EBX: base (not general purpose?) - can be used to store data
- ECX: counter used for iterations (counts downward)
- EDX: an extension of the EAX register 
  - for more complex operations such as multiply, divide by allowing extra data to be stored to facilitate those calculations
- ESP: stack pointer - the top of the stack
- EBP: base pointer - used to mark the start of a function's stack frame
- ESI: source index - holds the location of input data
- EDI: destination index - points to the location where the result of data operation is stored
- EIP: instruction pointer

ESP and EBP are almost exclusively used to manage the entry and exit to a procedure. 

EBX ESI and EDI must be preserved; EAX ECX and EDX can be freely modifies within the procedure that is using them.

Although functions can access local variables by the offsets of ESP, a better practice would be to use a frame pointer that saves the current address of the stack to the EBP register. Fuctions can reference their local variables by using the offsets of EBP without worrying about clobbering the stack.

#### Stack Usage

##### Using ESP and EBP

```assembly
	call procname

procname:
	push ebp	;preserve base pointer
	mov esp,ebp	;stack pointer into ebp
				;for debugging aid & exception handling (in the cases of calling multiple functions)
	sub 8,esp   ;subtract space from stack for local variables
	
	;assembler code
	
	mov ebp, esp ;restore stack pointer
	pop ebp		 ;restore base pointer
	
	ret
label:

;address of parameters
;ebp
;address of local variables
```

#### Stack Based Buffer Overflows:

- affects any functions that copies input to memory without bounds checking
  - strcpy()
  - memcpy()
  - gets()
  - etc

- overflow: when the source data size is larger than the destination buffer size, it overflows the buffer towards higher memory address and overwrites previous data on the stack (as the stack grows towards lower address)

#### Attempts to mitigate exploits on buffer overflow:

- stack canaries
- Address Space Layout Randomization 

Links:

[Stack-based memory allocation](https://en.wikipedia.org/wiki/Stack-based_memory_allocation)

[Understanding & Exploiting stack-based Buffer Overflows](https://medium.com/@sghosh2402/understanding-exploiting-stack-based-buffer-overflows-acf9b8659cba)

[Stack based buffer overflow Exploitation Tutorial](https://www.exploit-db.com/docs/english/28475-linux-stack-based-buffer-overflows.pdf)

