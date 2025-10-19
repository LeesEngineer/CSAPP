</br>

# MachineLevelProgramming

</br>

<p>Nowadays, what the target of a compiler is to generate assembly code. When I say machine code, I sort of interchangeably mean object code the binary form or assembly code the text version of it</p>

</br>

# Assembly code view

</br>

<img width="1708" height="624" alt="QQ_1760860253238" src="https://github.com/user-attachments/assets/e1125ce0-92c3-4bb4-af8a-857a36ca3eff" />

<p>Programmer-visible State</p>

- PC (Program counter)

  1. Address of next instruction
 
  2. Called "RIP" (x86-64)
 
- Register file

  1. Heavily used program data
 
- Condition codes

  1. Store status information about most recent arithmetic or logical operation
 
  2. Used for conditional branching
 
- Memory

  1. Byte addressable array
 
  2. Code and user data
 
  3. Stack to support procedures

<p>The memory is you can think of logically as just an array of bytes. And that's what the machine-level programmers see. It's actually kind of a fiction in different ways. That's sort of a collaboration between the operating system and hardware. What they call virtual memory to make it looks like each program running on the processor has its own independent array of bytes that it can access</p>

<p>Even though they actually share values within the physical memory.</p>

<p>Furthermore, you heard the term "cache". The idea of cache is not visible here at all. Because it just is automatically loaded with recent stuff. And the only thing that will look different is if you re-access that memory it will go faster</p>

<p>But cache is not visible in terms of program. There is no instructions to manipulate the cache. There's no way you can directly access the cache. </p>

</br>

# Turning C to into object code

</br>

<p>It's a many layered set of activities.</p>

<img width="1976" height="966" alt="QQ_1760861964699" src="https://github.com/user-attachments/assets/ca157576-8103-41b5-bb7b-27d22852c744" />

<p>There's actually some libraries that get imported dynamically when the program first begins.</p>

<p>Here is a not very intersting function in terms of doing anything useful, but it sort of demonstrates the basic ideas of compilation</p>

```
// C code
long plus(long x, long y);

void sumstore(long x, long y, long *dest)
{
    long t = plus(x, y);
    *dest = t;
}
```

<p>If I run it through a C compiler.</p>

```
// Generated x86-64 assembly
sumstore:
    pushq %rbx
    movq %rdx, %rbx
    call  plus
    moveq %rax, (%rbx)
    popq  %rbx
    ret
```

<p>Obtain with command (On shark machine) : `gcc -Og -S sum.c`</p>

<p>This is actuallt a slightly cleaed up version of what really happens.</p>

```
x                     -> %rdi
y                     -> %rsi
dest                  -> %rdx
result of plus        -> %rax
temporarily save dest -> %rbx
```

</br>

# Assembly Characteristics

</br>

</br>

## Data Types

</br>

- "Integer" data of 1, 2, 4, or 8 bytes. <b>They don't distinguish sign versus unsigned in how it gets stored. </b>

  1. Data values
 
  2. Addresses (Untyped pointers). Pointer is just stored as a number.
 
- Floating point data of 4, 8, or 10 bytes. A floating point hands with a different set of registers

- Code : Byte sequences encoding series of instructions

- No aggergate types such as arrays or structures. <b>Just contiguously allocated bytes in memory.</b>

<p>Things that you think of as fundamental datatypes, don't exist at the machine level. They're sort of constructed artifically by the compiler</p>

</br>

## Operations

</br>

<p>The other thing about machine level programming is each instruction is very limited in what it can do. It basically only do one thing.</p>

- Perform arithmetic function on register or memory data

- Transfer data between memory and register

  1. Load data from memory to register
 
  2. Store register data into memory
 
- Transfer control

  1. Unconditional jumps to/from procedures
 
  2. Conditional branches
 
</br>

# Disassembly

</br>

<img width="1872" height="666" alt="QQ_1760871148073" src="https://github.com/user-attachments/assets/4f51ec35-52f5-47ae-95aa-f85ad4911ad4" />

<p>Disassembler: `objdump -d sum > sum.d `. Can be run on either a.out(complete executable) or .o file</p>

<p>`gdb sum`</p>

<img width="1194" height="540" alt="QQ_1760871686672" src="https://github.com/user-attachments/assets/690fed1c-ae3f-4135-9166-821dd10eb573" />

</br>

## Assembly Basics

</br>

</br>

## x86-64 Integer Registers

</br>

<img width="1656" height="1184" alt="QQ_1760872199643" src="https://github.com/user-attachments/assets/ae6ae293-9061-4f75-b240-0c479118e2c7" />

<p>Can reference low-order 4 bytes (also low-order 1 & 2 bytes)</p>

<p>There's one special register -- %rsp, and that's called the stack pointer. It has a very specific puepose. There's some that are slightly different than the other, Fror the most part they are all usable for holding program data.</p>

<p>But just in case you're wondering why there's these weird names for these registers, you appreciate the fact that this is a legacy thing</p>

</br>

## Moving Data

</br>

<p>There're three kinds of mov, because it can take different types of information or they call operands. </p>

<img width="1570" height="988" alt="QQ_1760873325727" src="https://github.com/user-attachments/assets/2b2742fe-8d91-491b-bb4e-d88359d92cc2" />

<p>Just for the sake of convenience for the hardware designers, it doesn't let you directly copy from one memory location to another.</p>































                                                      
                           
                           
                           
                           
                           
                           


































































































































































































































































































































































































































































































































































































































































































































































































