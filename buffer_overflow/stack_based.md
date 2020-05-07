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

#### Basics in Assembly Language

- registers are not case-sensitive 

##### move (Opcodes: 88, 89, 8A, ...): 

- copies the data item referred to by its first operand into the location referred to by its second operand

- register-to-register moves are possible, direct memory-to-memory moves are not

- the source memory contents must first be loaded into a register, then can be stored to the destination memory address

- ```assembly
  ;Syntax
  mov <reg>,<reg>
  mov <reg>,<mem>
  mov <mem>,<reg>
  mov <const>,<reg>
  mov <const>,<mem>
  
  ;Examples
  mov [ebx], eax		;move the 4 bytes in memory at the address contained in ebx into eax
  mov cl, [esi+eax]	;move the contents of cl into the byte at address esi+eax
  mov 1h, al			;load al with immediate value 1
  mov ds, dx			;move the contents of dx into segment register ds
  ```

##### push (Opcodes: FF, 89, 8A, 8B, ...):

- places its operand onto the top of the hardware supported stack in memory

- specifically, push first decrements ESP by 4 (the stack pointer going down as the stack grows towards lower addresses), then places its operand into the contents of the location at address [ESP]

- ```assembly
  ;Syntax
  push <reg32>
  push <mem>
  push <con32>
  
  ;Examples
  push eax 	;push eax on the stack
  push [var] 	;push the 4 bytes at address var onto the stack
  ```

##### pop 

- removes the 4-byte data element frm the top of the hardware-supported stack into the specified operand (i.e. register or memory location)

- first moves the 4 bytes located at memory location [esp] into the specified register or memory location, and then increments esp by 4

- ```assembly
  ;Syntax
  pop <reg32>
  pop <mem>
  
  ;Examples
  pop edi		;pop the top element of the stack into EDI.
  pop [ebx]	;pop the top element of the stack into memory at the four bytes starting at location EBX. 
  ```

##### call

- a subroutine call
- first pushes the current code location onto the hardware supported stack in memory, and then preforms an unconditional jump to the code location indicated by the label operand
- unlike the simple jump instructions, the call instruction saves the location to return to when the subroutine completes

- ```assembly
  call <label>
  ```

##### ret

- pops a code location off the hardware supported in-memory stack, and then performs an unconditional jump to the retrieved code location

- ```assembly
  ret
  ```

#### Stack Usage:

##### Using ESP and EBP

```assembly
	call procname

procname:
	push ebp	;preserve base pointer
	mov esp,ebp	;moves stack pointer into ebp, ebp will act as the frame pointer
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

##### An example following complete calling convention:

```assembly
;from virginia cs
;different oprand orders in mov
.486
.MODEL FLAT
.CODE
PUBLIC _myFunc
_myFunc PROC
  ; Subroutine Prologue
  push ebp     ; Save the old base pointer value.
  mov ebp, esp ; Set the new base pointer value.
  sub esp, 4   ; Make room for one 4-byte local variable.
  push edi     ; Save the values of registers that the function
  push esi     ; will modify. This function uses EDI and ESI.
  ; (no need to save EBX, EBP, or ESP)

  ; Subroutine Body
  mov eax, [ebp+8]   ; Move value of parameter 1 into EAX
  mov esi, [ebp+12]  ; Move value of parameter 2 into ESI
  mov edi, [ebp+16]  ; Move value of parameter 3 into EDI

  mov [ebp-4], edi   ; Move EDI into the local variable
  add [ebp-4], esi   ; Add ESI into the local variable
  add eax, [ebp-4]   ; Add the contents of the local variable
                     ; into EAX (final result)

  ; Subroutine Epilogue 
  pop esi      ; Recover register values
  pop  edi
  mov esp, ebp ; Deallocate local variables
  pop ebp ; Restore the caller's base pointer value
  ret
_myFunc ENDP
END
```

![img](https://www.cs.virginia.edu/~evans/cs216/guides/stack-convention.png)

<div align="center">Stack during subroutine call </div>
<div align="center">credits:[x86 Assembly Guide](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html) </div>

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

[Assembly Language](https://en.wikipedia.org/wiki/Assembly_language#Assembler)

[x86 Assembly Guide](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html)

