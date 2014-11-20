---
---



**Cdecl** calling convention - The user calling the function is responsible for cleaning up the stack  
This is the default calling method in C  
It can also be manually specified with the `__cdecl` keyword  
i.e.: space on the stack for arguments must be de-allocated __after the function returns__

New function calls start with 
```
push ebp
mov ebp, esp
```
This is to:
1. preserve the current `ebp` (base pointer) by pushing it to the stack
2. preserve the current `esp` (stack pointer) by storing it in `ebp`

The reverse of this process is done after the function is finished, to return the stack, `esp`, and `ebp` to their original states.

Arguments to a function are pushed to the stack before calling it, and then removed (sort of) after. EG below:

The function in C:
```C
int func(int a, int b){
    //do something
}
```
The call to the function in assembly:
```x86
    push ebx ;assume ebx contains argument b
    push eax ;assume eax contains argument a
    call func ;call the function
    add esp, 8; two parameters on stack at 4 bytes each
```

The value of `esp` is the same at the beginning and the end of the above code block.


**stdcall** convention - parameters added to stack before a call are popped __before__ the called function returns.  
call using `__stdcall` keyword in function declaration  
**important:** there needs to be some communication regarding how many parameters are being passed or the location of `esp` may be unexpected

**fastcall** convention - first 2 parameters are stored in ecx and edx  
This is faster because you don't have to do 4 memory accesses per function call  
Call using `__fastcall` keyword in function declaration  
Less of a mess on the stack but then the called function must allocate more space on the stack (can do in one op eg: `sub esp, 14h` for five vars).  
Additional parameters beyond the first two are pushed to the stack.
Stack clean-up is done by called function before return eg: `ret 4` if one parameter is on stack

####Endianity/Endianness

*How data is represented in memory*  
Data in memory can be represented with the Least-Significant Byte (LSB) at the beginning or at the end of it.  
eg: Big endian for 1:
```
00 00 00 00 00 00 00 01
```
little endian for 1:
```
01 00 00 00 00 00 00 00
```

   *note*: There are other ways to do it, but they're weird(er) and complicated and we probably won't see them today.

####Challenges:

1. Write a function that takes two parameters and changes the first one without referencing it. eg:  
    ```c
    int func(int a, int b){
        //Change the value of a without referencing either parameter
    }
    ```
2. Repeat exercise 1, but with a function that uses the fastcall convention below:  
    ```c
    int __fastcall func(int a, int b){
        //Change the value of a without referencing either parameter
    }
    ```
3. Print all the opcodes (instructions) in main
4. What's the difference between the two structs below:  
    struct 1:  
    ```c
    struct s
    {
        int a;
        int b;
    }
    ```
    struct 2:  
    ```c
    struct t
        {
        int a;
        int b;
        int c;
        int d;
        int e;
        int f;
    }
    ```
5. Print the addresses of all of the 'call' opcodes inside main.

   note: stack operations and calling conventions in asm seem to be a source of confusion today -- consider this for a lecture later on.  
   other important topics:  
   - Structure of memory
   - Direction of Stack growth
   - Size of registers
   - Size of primitive data types

