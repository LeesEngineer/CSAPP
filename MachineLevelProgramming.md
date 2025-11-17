</br>

## MachineLevelProgramming

</br>

<p>Nowadays, what the target of a compiler is to generate assembly code. When I say machine code, I sort of interchangeably mean object code the binary form or assembly code the text version of it</p>

</br>

## Assembly code view

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

## Turning C into object code

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

<p>ret gets you return from wherever the calling position was. </p>

<p>This is actuallt a slightly cleaed up version of what really happens.</p>

```
x                     -> %rdi
y                     -> %rsi
dest                  -> %rdx
result of plus        -> %rax
temporarily save dest -> %rbx
```

</br>

## Assembly Characteristics

</br>

</br>

### Data Types

</br>

- "Integer" data of 1, 2, 4, or 8 bytes. <b>They don't distinguish sign versus unsigned in how it gets stored. </b>

  1. Data values
 
  2. Addresses (Untyped pointers). Pointer is just stored as a number.
 
- Floating point data of 4, 8, or 10 bytes. A floating point hands with a different set of registers

- Code : Byte sequences encoding series of instructions

- No aggergate types such as arrays or structures. <b>Just contiguously allocated bytes in memory.</b>

<p>Things that you think of as fundamental datatypes, don't exist at the machine level. They're sort of constructed artifically by the compiler</p>

</br>

### Operations

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

### Disassembly

</br>

<img width="1872" height="666" alt="QQ_1760871148073" src="https://github.com/user-attachments/assets/4f51ec35-52f5-47ae-95aa-f85ad4911ad4" />

<p>Disassembler: `objdump -d sum > sum.d `. Can be run on either a.out(complete executable) or .o file</p>

<p>`gdb sum`</p>

<img width="1194" height="540" alt="QQ_1760871686672" src="https://github.com/user-attachments/assets/690fed1c-ae3f-4135-9166-821dd10eb573" />

</br>

### Assembly Basics

</br>

</br>

### x86-64 Integer Registers

</br>

<img width="1656" height="1184" alt="QQ_1760872199643" src="https://github.com/user-attachments/assets/ae6ae293-9061-4f75-b240-0c479118e2c7" />

<p>Can reference low-order 4 bytes (also low-order 1 & 2 bytes)</p>

<p>There's one special register -- %rsp, and that's called the stack pointer. It has a very specific puepose. There's some that are slightly different than the other, Fror the most part they are all usable for holding program data.</p>

<p>But just in case you're wondering why there's these weird names for these registers, you appreciate the fact that this is a legacy thing</p>

</br>

### Moving Data

</br>

<p>There're three kinds of mov, because it can take different types of information or they call operands. </p>

<img width="1570" height="988" alt="QQ_1760873325727" src="https://github.com/user-attachments/assets/2b2742fe-8d91-491b-bb4e-d88359d92cc2" />

<p>Just for the sake of convenience for the hardware designers, it doesn't let you directly copy from one memory location to another.</p>

</br>

### Memory Addressing Modes

</br>

- Normal : (R)  Mem[Reg[R]]

  1. Register R specifies memory address
 
  2. Pointer dereferencing in C
 
  3. movq (%rcx), %rax

- Displacement D(R)  Mem[Reg[R] + D]

  1. Register R specifies start of memory region
 
  2. Constant displacement D specifies offsets
 
  3. movq 8(%rbq), %rdx

<p>In front of this parenthesis it means to offset not to use the address that's in the register. But add or subtract some number from it. To get an address that's just slightly off it.</p>

- Most General Form : D(Rb, Ri, s)  Mem[Reg[Rb] + S * Reg[Ri] + D]

  1. D : Constant displacement 1, 2, or 4 bytes
 
  2. Rb : Base register : Any of 16 integer registers.
 
  3. Ri : Index Register : Any, except for %rsp.
 
  4. S : Scale factor : 1, 2, 4, or 8
 
<p>It turns out this will be the sort of natural way to implement array referencing. Basically you can think of is if this is an array index, I have to typically scale it by however many bytes my data type is. If it's an int I have to scale it by four. If it's a long I have to scale it by eight. </p> 

<img width="1678" height="618" alt="QQ_1760940915146" src="https://github.com/user-attachments/assets/6f9fa54b-e3fa-47ea-91b2-7ce39cff84db" />

</br>

### Address Computation Instruction

</br>

- leaq Src, Dst (Load Effective Address (Qword))

  1. Src is address mode expresion.
 
  2. Set Dst to address denoted by expression.
 
- Uses

  1. Computing address without a memory reference. E.g., translation of p = &x[i];
 
  2. computing arithmetic expressions of the form x + k * y. k = 1, 2, 4, or 8.
 
<p>Example:</p>

```
long m12(long x)
{
    return x * 12;
}

//Converted to ASM by compiler
leaq (%rdi, %rdi, 2), %rax
salq $2, %rax
```
 
<p>leaq basically use the ampersand operation of C to compute an address. It also turns out to be a pretty handy way to do arithmetic and the C compiler likes to use it.</p>

</br>

### Some Arithmetic Operations

</br>

<img width="1122" height="782" alt="QQ_1760942613666" src="https://github.com/user-attachments/assets/4763bb90-6a41-4a48-90f7-7d749ff0fe38" />

<p>shrq is logical shift right. sarq is arithmetic shift right.</p>

<p>No distinction between signed and unsigned int.</p>

<img width="1012" height="354" alt="QQ_1760943975551" src="https://github.com/user-attachments/assets/55f8e610-9e1c-47ca-946b-50ef9965ab11" />

</br>

# Chapter Two Control

</br>

<p>Information about currently executing program.</p>

- Temporery data (%rax, ...)

- Location of runtime stack (%rsp)

- Location of current code control point (%rip, ...)

- Status of recent tests (CF, ZF, SF, OF)

<p>There's another register that they call the %rip. It means instruction pointer. All it contains is the address of currently executing instruction. It's not a register that you can access in a normal way. It just tells you where in the program. </p>

</br>

## Condition Codes

</br>

<p>The x84 and several machine of its generation have these curious little of one bit flags that are called condition codes. They are the basis of how conditional operations works</p>

<p>CF means the carry flag (For unsigned). The ZF (zero flag) is what it sounds like it's set if the value you just computed is zero. The SF (sign flag for signed) is said if the value just computed as a one in the most significant bit meaning it's a negative value. And the OF (over flag) is a two's complement version of overflow . These are set typically by arithmetic instructions.</p>

<p>These four flags get set as a sort of normal activity by many of the instructions, not by our friend the lea instruction, which I mentioned is kind of a quirky instruction that GCC really likes a lot.</p>

<p>There is some special instructions whose only effect is to set condition codes : </p>

<p><b>Explicit Setting by Compare Instruction.</b></p>

- cmpq Src2, Src1

- cmpq b, a like computing a - b without setting destination

- CF set if carry out from most significant bit (used for unsigned comparisons)

- ZF set if a == b

- SF set if (a - b) < 0 (as signed)

- OF set if two's complement (signed) overflow

<p><b>Explicit Setting by Test Instruction.</b></p>

<p>The only purpose of test is to set condition flags.</p>

<p>When you have two values and you want to compare them with each other, you'd like use cmpq. And the test is if you just have one value and you want to see what it's like is (zero? negative? ...).</p>

- test Src2, Src1

- test b, a like computing a & b without setting destination

- Sets condition codes based on value of Src1 & Src2

- Useful to have one of the operands be a mask

- ZF set when a & b == 0

- SF set when the most significant bit is one ((a & b) < 0)

<p>What you'll typically see is a test where both arguments are the same so testq %rax, %rax</p>

</br>

### Reading Condition Codes

</br>

<p>Setx instructions : Set low-order byte of destination to 0 or 1 based on combinations of condition codes. Does not alter remaining 7 bytes</p>

<img width="1472" height="866" alt="QQ_1761199098281" src="https://github.com/user-attachments/assets/eaff1450-a878-4ce6-81b6-f493a2b2b4bc" />

<p>Can referance low-order byte.</p>

<img width="1826" height="1168" alt="QQ_1761200922242" src="https://github.com/user-attachments/assets/46182c4b-c6d3-4940-a442-5ea8939ec67f" />

<b>one of addressable byte registers : </b>

- Does not alter remaining bytes

- Typically use movzbl to finish job <b>(32-bit instructions also set upper 32 bits to 0)</b>

```
int gt(int x, int y)
{
    return x > y;
}
```

```
cmpq %rsi, %rdi
     setg %al
     movzbl %al, %eax  # Set when >
     ret               # Zero rest of %rax
```

<p>%rdi is used as the first parameter, and %rsi is the second. %rax is used as the return value.</p>

</br>

## Branches

</br>

<b>Jumping</b>

<p>jx instructions : Jump to different part of code depending on condition codes.</p>

<img width="1472" height="944" alt="QQ_1761203848619" src="https://github.com/user-attachments/assets/d00afe1b-6f51-4ae6-ad34-c509e3ed63bc" />

<p>Example : </p>

```
long absdiff(long x, long y)
{
    long result;
    if(x > y)
        result = x - y;
    else
        result = y - x;
    return result;
}
```

```
absdiff:
    cmpq    %rsi, %rdi
    jle     .L4
    movq    %rdi, %rax
    subq    %rsi, %rax
    ret
.L4:
    movq    %rsi, %rax
    subq    %rdi, %rax
    ret
```

<p>In general in assembly code if you give a name and then a colon, what's to the left of that is called a label</p>

<b>Conditional Move Example : </b>

```
absdiff:
movq    %rdi, %rax
subq    %rsi, %rax
movq    %rsi, %rdx
subq    %rdi, %rdx
cmpq    %rsi, %rdi
cmovle  %rdx, %rax
ret
```

<p>It's like this idea of just go ahead and do everything and then pick at the last end.</p>

<p>Bad Cases for Conditional move</p>

- Expensive Computions : `val = Test(x) ? Hard1(x) : Hard2(x);`

  1. Both values get computed.
 
  2. Only makes sense when computations are very simple

- Risky Computation `val = p ? *p : 0;`

  1. Both values get computed
 
  2. Many have undesirable effects
 
- Computations with side effects `val = x > 0 ? x *= 7 : x += 3;`

  1. Both values get computed
 
  2. Must be side-effect free
 
</br>

## Loops

</br>p

<p>C actually has three different kinds of loops, it has a while-loop and a for-loop which you're familiar with, and dowhile-loop. But it turns out it's the simplest one to implement.</p>

</br>

### do-while loop

</br>

<p>If we think in terms of goto, that's a pretty straightforward thing to replace it. </p>

```
long pcount_do(unsigned long x)
{
    long result = 0;
    do
    {
        result += x & 0x1;
        x >>= 1;
    } while(x);
    return result;
}
```

```
long pcount_goto(unsigned long x)
{
    long result = 0;
loop:
    result += x & 0x1;
    x >>= 1;
    if(x) goto loop ;
    return result;
}
```

```
    movl    $0, %eax
.L2:
    movq    %rdi, %rdx
    andl    $1, %eax
    addq    %rdx, %rax
    shrq    %rdi
    jne     .L2
    rep; ret
```

</br>

### while loop

</br>

```
while (Test)
    Body
```

```
    goto test
loop;
    Body
test:
    if(Test)
        goto loop;
done; 
```

- "Jump to middle" translation

- Used with -Og 

<p>There is two ways to generate a code for while-loop. Nut you'll find GCC actually uses two different ways. A One is using -Og. O stands for optimized, g which means debug. As I mentioned last time this turns out to be the perfect level of optimization.</p>

<p>Where you want to be able to look at machine code and understand it and how it relates to the c code. Because it does some sort of simple optimizations. But it doesn't try to rewrite your whole program to make it run better.</p>

<p>Whereas even with -O1, which is the next level in the optimization. You will find sometimes it will do some pretty quirky stuff.</p>

<p>So usually there's higher levels optimization, and we are purposely backing off from that to make this code easier tounderstand.  </p>

</br>

### For Loop

</br>

<p>For loop has four components. It has an initialization, has a test, has a rule for doing an update in case as a way to continue the loop, and then it has a body of the loop.</p>

```
#define WSIZE 8 * sizeof(int)
long pcount_for(unsigned long x)
{
    size_t i;
    long result = 0;
    for(i = 0; i < WSIZE; i ++)
    {
        unsigned bit = (x >> 1) & 0x1;
        result += bit;
    }
    return result;
}
```

<p>For loop -> while loop</p>

```
for(Init; Test; Update)
    Body

Init;
while(Test)
{
    Body
    Update;
}
```

</br>

## Switch statements

</br>

```
long switch_eg(long x, long y, long z)
{
    long w = 1;
    switch(x)
    {
    case 1:
        w = y * z;
        break;
    case 2:
        w = y / z;

    case 3:
        w += z;
        break;
    case 5:
    case 6:
        w -= z;
        break;
    default:
        w = 2;
    }
    return w; 
}
```

- Multiple case labels : 5 & 6

- Fall through cases : 2

- Missing cases : 4

<p>If you were told not use switch statements anymore, what you'd probably do is write a big long chain of if-else. You'd expect this to be the machine code of switch statement, but it's not. let me show you what the machine code does. </p>

<img width="2126" height="788" alt="QQ_1761640046101" src="https://github.com/user-attachments/assets/c2791a79-1929-4000-9e5a-ad3df304fe9c" />

<p>So think of the general form of it as being some blocks of code. The entry points of which are labeled by these case values. And then the box string together in various different ways and do various things. What I'm going to do is compile a code for all of those blocks, and store them away in some part of memory, load up memory to contain these code blocks.</p>

<p>I build a table, and each entry of this table describes the starting location of one of these code blocks. This table will have many entries of addresses to tell me where these code are located.</p>

<p>There's a cool instruction. It's like a 'ray indexing'. If you think of a ray indexing, it means you can grab a value out of the middle of array or some set of values you know. without having to step through them one by one.</p>

<p>It's the same idea here that I will take my value and use that to figure out directly where I should jump to a block of code without having to step through a branch of other conditions(chain of if-else). You can see the difference in efficiency between sort of in one step knowing exactly where you want to be versus stepping through you know on average n over two conditions to get to where I want to go</p>

<p>Let's look at this at the assembly code level</p>

```
long switch_eg(long x, long y, long z)
[
    long w = 1;
    switch(x)
    {
        ...
    }
    return w;
]
```

```
switch_eg:
    movq    %rdx, %rcx
    cmpq    $6, %rdi
    ja      .L8
    jmp     *.L4(, %rdi, 8)
```

<p>It's just making a copy of argument z here for some reason. And then it's comparing x to 6(Because 6 was the largest value of any of cases.). And it's using a jump instruction to go to .L8 , what we'll find is that tells you what the default behavior should be. But also it will cause it to jump if x is less than 0.</p>

<p>Then the final part is the real heart of the work. This is a very special goto instruction. That lets me index into a table and extract out of that an address, and then jump to that address. So that's what lets me go directly up to some block of code, based on whether the values will be in the range between 0 and 6.</p>

<p>Each entry is 8 bytes, which is the size of one address. The asterisk(*) here indicates an indirect jump, meaning: Instead of jumping directly to a fixed label, it jumps to an address stored in a register or memory.(.L4 will be interpreted as a value.)</p>

<p><b>Jump table</b></p>

```
.section    .rodate 
  .align 8
.L4:
    .quad   .L8  # x = 0
    .quad   .L3  # x = 1
    .quad   .L5  # x = 2
    .quad   .L9  # x = 3
    .quad   .L8  # x = 4
    .quad   .L7  # x = 5
    .quad   .L7  # x = 6
```

<p>It's specified in assembly code. And it's the job of the assembler to actually fill in the contents of the table in binary file. Meaning the jump if you look at the .s file the assembly code file, it's all in there and the compiler generated these tables(at least the sort of framework for these tables). But the details of which get filled in by assembler</p>

<p>I need a quad is just a declaration to say I need an 8 byte value here, and that value should match whatever address.</p>

- Table Structure

  1. Each target requires 8 bytes
 
  2. Base address at .L4. It indicates the address of the memory location starting from here.

<img width="1720" height="1126" alt="QQ_1761722227670" src="https://github.com/user-attachments/assets/5a826bf0-1033-4523-a91b-1b294567fbc9" />

<p>We can see some of the logic of this switch statement.</p>

<hr>

<p>Now, the rest od it is to look at the various code box.</p>

</br>

### x == 1

</br>

```
switch(x)
{
case 1 : // .L3
    w = y * z;
    break;
  ...
}
```

```
.L3:
    movq    %rsi, %rax
    imulq   %rdx, %rax
    ret
```

<p>Break is just going to be turned into ret instruction here. The compiler doesn't actually come to a single point and say okay everyone returned at this point. It just sticks returns directly in wherever these breaks occur. </p>

</br>

### Handling Fall-Through

</br>

<p>Here is actually a curious by the way I'm always somewhat surprised by what the compiler does. </p>

<img width="1930" height="1058" alt="QQ_1761726263765" src="https://github.com/user-attachments/assets/e7601b50-b2b1-488e-a3b8-4a27967a6271" />

<p>In particular, It patched together the pair of fall through case. And it had to do these separately. Because remember w was not set before I entered these code blocks. It deferred setting. And here I hit case three and all of a sudden I actually need whatever w was which was one. so the compiler will set w to 1 here before we continue. So as a result it sort of creates two code blocks. But case two will jump from it. </p>

```
long w = 1;
  ...
switch(x)
{
  ...
case 2:
    w = y / z;
case 3:
    w += z;
    break;
  ...
}
```

```
.L5:
    movq    %rsi, %rax
    cqto
    idivq   %rcx
    jmp     ,L6
.L9:
    movl    $1, %eax
.L6:
    addq    %rcx, %rax
    ret
```

<p>Very quirky</p>

<p>Once again it's making use of this feature that movl will set the upper 32 bits to zero.</p>

</br>

### x == 5, x == 6, default

</br>

```
switch(x)
{
  ...
case 5:
case 6:
    w -= z;
    break;
default:
    w = 2;
}
```

```
.L7:
    movl    $1, %eax
    subq    %rdx, %rax
    ret
.L8:
    movl    $2, %eax
    ret
```

<p><b>On a 64-bit sysytem, if you just only assign a small constant to %rax, using movl is smaller and faster.</b></p>

<b>Summarizing:</b>

- C control

  1. if-then-else
 
  2. do-while
 
  3. while, for
 
  4. switch
 
- Assembler control

  1. Conditional jump
 
  2. Conditional move
 
  3. Indirect jump (via jump tables)
 
  4. Compiler generates code sequence to implement more complex control
 
- Techniques

  1. Loops converted to do-while or jump-to-middle form
 
  2. Large switch statements use jump tables
 
  3. Sparse switch statements may use decision tree(if-elseif-elseif-else)

</br>

# Chapter Three Procedures

</br>

<p>At the assembly level, a procedure is a block of code that is called, executing and returning</p>

<p>If you think about what goes on in a procedure. E ven in C which is a relatively unsophisticated language in many ways, ther's a lot going on.</p>

</br>

## Mechanisms in Procedures

</br>

<p>To implement this, the compiler needs the support of several underlying mechanisms.</p>

- Passing control

  1. To beginning of procedure code
 
  2. Back to return point
 
- Passing data

  1. Procedure arguments
 
  2. Return value
 
- Memory management

  1. Allocate during procedure execution
 
  2. Deallocate upon return
 
<p>Mechanisms all implemented with machine instructions. The compiler translates the function you write into a series of assembly instructions.</p>

<p>x86-64 implementation of a procedure uses only those mechanisms required. Because the compiler is smart, it doesn't always build a full stack frame. For example, if a function has no local variables, doesn't call other funcations, and doesn't need to save any registers, the compiler even omit the usual prologue and epilogue instructions such as "push", "mov %rsp, %rbp" and "pop". They do whatever is absolutely needed. </p>

</br>

## Stack Structure

</br>

<p>The first one and sort of the most critical is how do we pass control to a function. Before wo can talk about that we have to talk about the stack.</p>

- Region of memory managed with stack discipline

- Grows toward lower addresses

- Register %rsp (stack pointer) contains lowest stack addess.<b>(Address of "top" element)</b>

<img width="1142" height="1228" alt="QQ_1761900386221" src="https://github.com/user-attachments/assets/e0137ac2-ea87-4d60-bdce-27591bc14e4a" />

<p>Stack is used by the program to manage the state associated with the procedures that it calls and as they return. So it's where program passes all these potential information, the control information, data and allocates local data. It makes use of that sort of last in first out allocation principle, matches very well with this idea of procedure call and return.</p>

<p>In x86 stacks, actually start with a very high numbered address. And when they grow more data are allocated for the stack, It's done by decrementing the stack pointer. It's value is the address of current top of the stack. And every time you allocate more space on the stack it does by decrementing that pointer.</p>

<p>There are explicit instructions push and pop that make use of the stack.</p>

- pushq Src

  1. Fetch operand at Src. Src could be from register, memory or an immediate
 
  2. Decrement %rsp by 8
 
  3. Write operand at address given by %rsp
 
  4. That address of memory is determined by first decrementing the stack pointer and then doing a write
 
- popq Dest

  1. Read value at address given by %rsp
 
  2. Increment %rsp by 8
 
  3. Store value at Dest(must be register)
 
<p>One thing you'll notice here is when I say deallocate, it's not meaning I magically erase this or something all I'm doing is just moving a stack pointer. Whatever it was there at the top of the stack is still in memory.</p>

</br>

## Passing Control

</br>

<p>The instructions push and pop are to put data on the stack or take it off. We use the same basic idea for a call and return.</p>

```
void multstore(long x, long y, long *dest)
{
    long t = mult2(x, y);
    *dest = t;
}
```

```
0000000000400540 <multstore>:
400540:  push   %rbx            # Save %rbx
400541:  mov    %rdx,%rbx       # Save dest
400544:  callq  400550 <mult2>  # mult2(x,y)
400549:  mov    %rax,(%rbx)     # Save at dest
40054c:  pop    %rbx            # Restore %rbx
40054d:  retq                   # Return
```

```
long mult2(long x, long y)
{
    long s = x * y;
    return s;
}
```

```
0000000000400550 <mult2>:
400550:  mov    %rdi,%rax
400553:  imul   %rsi,%rax
400557:  retq
```

<p>Parameter passing rules (x86-64 system V ABI): </p>

<p>The first 6 integer (or pointer) parameters get passed within these particular registers (There is no particular logic to it): %rdi, %rsi, %rdx, %rcx, %r8 and %r9.(in order)</p>

<p>Why store %rdx in %rbx : We need to know the value of dest after mult2 return. however, %rdx is likely to be corrupted during mult2's call. Therefore, it must be saved beforehand.</p>

<p>And according to System V x86-64 ABI , registers are divided into two categories :</p>

- Caller-saved

  1. %rax, %rcx, %rdx, %rsi, %rdi, %r8–%r11
 
  2. May be modified
 
- Callee-saved

  1. %rbx, %rbp, %r12–%r15
 
  2. Must be restored to original value
 
<p>So %rbx is used to temporarily store dest.(register saving conventions)</p>

<p>register saving conventions prevent one function call from corrupting another's data. <b>Unless the C code explicitly does so(e.g., buffer overflow in Lecture 9)</b></p>

<hr>

<b>Procedure Control Flow : </b>

- Use stack to support procedure call and return

- Procedure call : call label

  1. Push return address on stack
 
  2. Jump to label
 
- Return address

  1. Address of the next instruction right after call
 
  2. Example from disassembly
 
- Procedure return : ret

  1. Pop address from stack
 
  2. Jump to address
 
<p>Keep in mind that call and ret don't do the whole business of the procedure call and return. They just do the control part of it which as we saw is only one of three aspects of a procedure</p>

<img width="2000" height="1276" alt="QQ_1761993941884" src="https://github.com/user-attachments/assets/15eeef6a-3f50-4538-9334-d220611d8ecc" />

<p>Let's break this down into its simplest part. Let's imagine a scenario. In which the top of stack is at 0x120.</p>

<p>And the program counter which is called %rip which is not anything to do with death(rest in peace). It's indicating that the current instruction is at 0x400544 which is this call instruction.</p>

<p>So what would happen with the call instruction, it would actually do three things. It would decrement the stack pointer and so subtracting 8 from (0x)120 and hex gives you (0x)118, and would write the address of the instruction following the call onto the top of the stack. That's the instruction that I'm going to return and used for my return address.(the instruction after the call instead of the call itself otherwise you'd have an infinite loop.)</p>

<p>The binary encoding of this call instruction also includes the destination address, which corresponds to the entry point of the particuler called function. So the program counter will be set to that value. Now the processor starts executing along these instruction. So it did a combination of a jump and a push. </p>

<p>You never manipulate the %rip register explicitly. It's implicitly part of the call instruction. But embedded in the call instruction you see it's five bytes long.(I don't show you the byte encoding)</p>

<p>When hit ret instruction, its purpose is to sort of reverse the effect of a call. It assume that the top of stack has an address that you want to jump to. It will pop that address off stack,(increment the stack pointer) and then it will set the program counter to what just popped off stack. That will cause the program to resume back to where it comes from. How clever the idea!</p>

<p>We use kind of combinations of instructions to build up all the layers associated with operations like procedure call and return.</p>

</br>

## Passing Data

</br>

<p>We've seen a couple registers that get used when you're passing arguments to a function and %rax get used to return values from a function. And again this is all built into ABI.</p>

<p>By the way this is all for arguments that are either integers or pointers, I've got a little bit on floating point those are passed in a separate set of registers.</p>

<p>What happens if you have more than 6 arguments to a function, well the rule on that is those get put in memory on stack</p>

<p>As long as everyone sticks to this common interface standard, then you can even use different compilers to compile code, and have them be able to cooperate with each other in terms of passing arguments and returning data.</p>

</br>

## Managing Local Data 

</br>

<p>What if there's some local data that we need to make use of.</p>

- Languages that support recursion

  1. e.g., C, Pascal, Java
 
  2. Code must be "reentrant" : Multiple simultaneous instantiations of single procedure. And these instantiations don't interfere with each other.
 
  3. Need some place to store states of each instantiations : Arguments, Local variables, Return pointer.
 
- Stack discipline

  1. State for given procedure needed only in limited time. From when called to when return
 
  2. Callee returns before caller does
 
- Stack allocated in Frames

  1. State for single procedure instantiation
 
<p>To get this idea, we introduce another concept whick is called the stack frame. A stack frame is essentially a specific pattern of memory allocation used during program execution. One of the key characteristics of function calls and returns is that, when a program performs a nested series of function calls, each invocation allocates its own distinct region on the stack — its stack frame.</p>

<p>When a particular function is executing, It only needs to reference the data within that function or values that have been passed to it. But the point is the rest of the functions in your code however many there are are sort of frozen at that moment, there's only one function executing at any given time.(Single threaded model here) So we can allocate on this stack whatever space is required for this particular function. And then when we return from that function, there's no need to preserve any of the information associated with the function.</p>

<p>That's why we use it. if you make more calls, you keep allocating more stuff. But as they return you back out of the stack and free things up.</p>

<p>So each block we use for a particular call is called the stack frame. To be more technical we'll say stack frame is a particular instantiation of a particular procedure.</p>

<b>Stack Frames(This is the same stack in which you pushing and poping addresses to)</b>

- Contents

  1. Return information
 
  2. Local storage(if needed)
 
  3. Temporary space(if needed)
 
- Management

  1. Space allocated when enter procedure
 
     - "Set-up" code
    
     - Includes push by call instruction
    
  2. Deallocated when return
 
     - "Finish" code
    
     - Includes pop by ret instruction

<p>One of the reasons why recursion is a little bit of risky thing : It keeps requiring more space as you go deeper in the recursion. And most systems limit the toatl depth of the stack.</p>

<img width="1054" height="1084" alt="QQ_1762147967397" src="https://github.com/user-attachments/assets/57c77bd4-84bb-4e35-bc09-4dcd6040416c" />

<p>In general stack frame is delimited by two pointers. One is the satck pointer %rsp which we're familiar with. the another calls the base pointer which %rbp indicates. But one feature is that this is an optional pointer. We don't use %rbp except in some very special cases.(It will be used instead just as a regular register)</p>

<p>Traditional(e.g., in earlier compilers or when compiling with -fno-omit-frame-pointer), %rbp acts like a fixed anchor point, telling us : </p>

- Where the stack frame begins

- How much local variables are offset

- How mush space is restored upon return

<p>whenever the function is called : </p>

```
push    %rbp
mov     %rsp, %rbp
sub     $N, %rsp
```

<p>Before the function returns : </p>

```
mov     %rbp, %rsp
pop     %rbp
ret
```

<p>The question here is if %rbp was optional then how does the program know how to do the deallocation. How can it reset the stack back to the right place: </p>

<p>The compiler knows the size of the stack frame when generating. Because the size of a static stack frame is fully known at compile time . For exampel, when it's going to allocate 16 bytes, and then it knows at the end of that it can deallocate 16 bytes.</p>

<p>There is a few special cases where it can't know in advance how much space will be allocated. When it has to allocate an array or a memory buffer of variable size(dynamic), it will use %rbp in these cases.</p>

<hr>

<p>x86-64/Linux stack frame : </p>

<p>Generally, the stack frame can be divided from top to bottom as follows.</p>

- Current stack frame ("Top to bottom")

  1. "Argument build"

     - Parameters for function about to call
    
     - The frirst six parameter may be passed through register, but the rest will be pushed onto the stack
    
     - This part is located at the top of the stack frame
 
  2. Local variables

     - The local variables that can't keep in registers such as array, some structure or those that compiler chooses to spill to the stack

  3. Saved register context

     - When another function is called，some registers will be modified
 
  4. Old frame pointer (Optional)
 
- Caller Stack Frame

  1. Return address : Pushed by call instruction
 
  2. Arguments for this call (if more than 6)

<img width="334" height="1296" alt="QQ_1762162641643" src="https://github.com/user-attachments/assets/fc5ff436-3ae8-4765-b1a3-250788a0787d" />

<p>Example:</p>

```
long incr(long *p, long val)
{
	long x = *p;
	long y = x + val;
	*p = y;
	return x;
}
```

```
incr:
	movq	(%rdi), %rax
	addq	%rax, %rsi
	movq	%rsi, (%rdi)
	ret
```

```
long call_incr()
{
	long v1 = 15213;
	long v2 = incr(&v1, 3000);
	return v1 + v2;
}
```

```
call_incr:
	subq	$16, %rsp
	movq	$15213, 8(%rsp)
	movl	$3000, %esi
	leaq	8(%rsp), %rdi
	call    incr
	addq	8(%rsp), %rax
	addq	$16, %rsp
	ret
```

<p>According to x84-64 System V ABI, before a function is called, the stack pointer %rsp must be 16-byte aligned. </p>

</br>

## Recursion

</br>

```
long pcount_r(unsigned long x)
{
	if(x == 0)
		return 0;
	else
		return (x & 1) + pcount_r(x >> 1);
}
```

```
pcount_r:
	movl	$0, %eax
	testq   %rdi, %rdi
	je		.L6
	pushq   %rbx
	movq	%rdi, %rbx
	andl	$1, %ebx
	shrq	%rdi
	call	pcount_r
	addq	%rbx, %rax
	popq	%rbx
.L6
	rep; ret 
```

<p>In general, recursive code is going to always generate a bigger blob of code than the iterative version.</p>

</br>

# Chapter Four Data

</br>

<p>Let's look at data in aggregated form, There's two way to do that. One is with arrays where you can create many copies of an identical data type. A second is where you have structs, so you create a small collection of values that can be different types.</p>

<p>The main thing is that at the machine code level there's no notion of an array except to think of it as a collection of bytes(in contiguous). And same with a struct</p>

</br>

## Arrays

</br>

<p>There is no bounds checking in C. So the compiler will happily let you use negative values for array indices, and it will give you a potentially undefined value. </p>

```
int get_digit(int z[5], int digit)
{
	return z[digit];
}
```

```
movl	(%rdi, %rsi, 4), %eax
```

```
void zincr(int z[5])
{
	size_t i;
	for(i = 0; i < 5; i ++)
		z[i] ++;
}
```

```
  movl	  $0, %eax
  jmp  	  .L3
.L4
  addl	  $1, (%rdi, rax, 4)
  addq	  $1, %rax
.L3
  cmpq    $4, %rax
  jbe	  .L4
  ret
```

<p>You see a pretty close correspondence here between instructions and machine code and constructs in the C programming</p>

<p>What's the real difference is between arrays and pointers in C.</p>

<P>When you declare an array in C, both you're actuallt allocating space and you are creating an array name that allows pointer arithmetic. Whereas when you just declare a pointer, all you're allocating is the space for the pointer itself.</P>

<img width="1960" height="338" alt="QQ_1762388189647" src="https://github.com/user-attachments/assets/c0e9f4a9-ccf9-454b-9231-d9539f1b2c3d" />

<p>Accesses looks similar in C, but address computations difffer greatly</p>

```
int get_univ_digit(size_t index, size_t digit)
{
	return univ[index][digit];
}
```

```
salq	$2, rsi
addq    univ(, %rdi, 8), %rsi
movl    (%rsi), %eax
ret
```

<hr>

<p>N * N Matrix Code</p>

- Fixed dimensions : Know value of N at compile time

```
#define N 16
typedef int fix_matrix[N][N]
int fix_ele(fix_matrix a, size_t i, size_t j)
{
	return a[i][j];
}
```

```
salq	$6, %rsi
addq	%rsi, %rdi
movl	(%rdi, %rdx, 4), %eax
ret
```

- Variable dimensions, explicit indexing : Traditional way to implement dynamic arrays

```
#define IDX(n, i, j) ((i) * (n) + (j))
int vec_ele(size_t n, int *a, size_t i, size_t j)
{
	return a[IDX(n, i, j)];
}
```

- Variable dimensions, implicit indexing : Now supported by gcc after C99

```
int var_ele(size_t n, int a[n][n], size_t i, size_t j)
{
	return a[i][j];
}
```

```
imulq   %rdx, %rdi
leaq	(%rsi, %rdi, 4), %rax
movl	(%rax, %rcx, 4), %eax
ret
```

<p>In the third example n is a parameter that passed to the function, So it's not known at compile time how big a scaling factor to use. You will see that it has to use a multiply instruction to do that, which is a relatively expensive instruction in terms of performance.</p>

<p>It used to be in C if you wanted to de multi-dimensional arrays, where the size of array was not fixed at compile time, you bassically had to implement your own version of the previous address computation. After C99, when calling a function you can pass in any integer value for n, and <b>the compiler will calculate the offset based on n.</b></p>

</br>

## Structures

</br>

- Structure represented as block of memory

  1. Big enough to hold all of the fields
 
- Fields ordered according to declaration

  1. Even if another ordering could yield a more compact representation
 
- Compiler determines overall size + positions of fields

  1. Machine-level program has no understanding of the structures in the source code

<p>Compiler will keep track of where each of these fields start, and generate the appropriate code to offset from the beginning, so the reference to structure is the beginning address of itself. Adnd then I'll use appropriate offset to get the different fields.</p>

```
struct rec
{
	int a[4];
	size_t i;
	struct rec *next;
};
```

```
int *getap(struct rec *r, size_t idx)
{
	return &r->a[idx];
}
```

```
leaq (%rdi, %rsi, 4), %rax
ret
```

<hr>

<p>Structures & Alignment</p>

```
struct S1
{
	char c;
	int i[2];
	double v;
} *p;
```

- Unaligned Data

<img width="1478" height="208" alt="QQ_1762500206964" src="https://github.com/user-attachments/assets/5864ccb9-aa91-4553-8051-2215bbd8fbb4" />

- Aligned Data
 
<img width="2000" height="548" alt="QQ_1762500297294" src="https://github.com/user-attachments/assets/156dcfea-8198-4f3d-b654-b2b421c79850" />

<p>The machine generally prefers that if you have a data type of K bytes, then the address that starts at will be a multiple of k. So that introduce a property we call alignment, when a structure gets allocated, the compiler will insert some blank unused bytes in the data structure.</p>

<p>Alignment Principles</p>

- Aligned Data

  1. Primitive data type requires K bytes
 
  2. Address must be multiple of K
 
  3. Required on some machines; advised on x86-64
 
- Motivation for Aligning Data

  1. Memory accessed by (aligned) chunks of 4 or 8 bytes (system dependent), If your data falls exactly within the block boundary, the CPU can retrieve it all in one go, otherwise, it will require two steps.
 
     - Inefficient to load or store datum that spans quad word boundaries
    
     - Virtual memory trickier when datum spans 2 pages
    
- Compiler

  1. Inserts gaps in structure to ensure correct alignment of fields

<p>On some machines, if you try to do an unwind access it will actually cause a memory fault.</p>

<hr>

<p>Overall Alignment Requirement (Arrays of structures)</p>

- For largest alignment requriement K

- Overall structure must be multiple of K

<p>Example : </p>

```
struct S2
{
	double v;
	int i[2];
	char c;
};
```

<img width="2030" height="200" alt="QQ_1762505954123" src="https://github.com/user-attachments/assets/81215e26-c598-4ff6-a234-f50103cc3442" />

<p>For this one, because it contains a double. The overall data structure has to be aligned on an 8 byte boundary.</p>

<p>It will add bytes to the end to make sure that the overall size of the data structure meets whatever underlying alignment requiremnet there is. </p>

<hr>

```
struct S3
{
	short i;
	float v;
	short j;
} a[10];
```

<img width="1886" height="454" alt="QQ_1762506432589" src="https://github.com/user-attachments/assets/c93cb798-60fb-4d05-8ad8-968e6d73b8e6" />

- Compute array offset 12 * idx

- Element j is at offset 8 within structure

- Assembler gives offset a+8 (Resolved during linking)

```
short get_j(int idx)
{
	return a[idx].j;
}
```

```
leaq	(%rdi, %rdi, 2), %rax
movzwl  a + 8(, %rax, 4), %eax
```

<hr>

<p>Saving space : Put large data types first</p>

<img width="1458" height="380" alt="QQ_1762507267911" src="https://github.com/user-attachments/assets/ab8f8754-e1b3-499c-8e48-aac719c89076" />

<p>Effect : </p>

<img width="922" height="260" alt="QQ_1762507312562" src="https://github.com/user-attachments/assets/71946855-08fa-4d88-9694-d85ebef63382" />

</br>

## Floating Point

</br>

<p>XMM registers are a special class of registers used for floating-point and vector calculations in the x86-64 architecture.</p>

<p>The XMM is a set of 128-bit registers (16 bytes) used in the SSE instruction set (Streaming SIMD Extensions).</p>

<p>SSE is an instruction extension introduced by Intel during the Pentium III era to accelerate:</p>

- Floating-point calculations (float/double)

- Vector operations (such as image processing and signal processing)

- Matrix calculations (such as in deep learning or computer graphics)

<img width="1846" height="1348" alt="QQ_1762508389822" src="https://github.com/user-attachments/assets/86c77dc9-e240-4db2-89af-d77afab0d260" />

<p>There are 16 special registers distinct from the other registers. Then there're operations that can operate on these and treat them in different ways. One is to treat such a register as an array of 16 chars or as 8 shorts or 4 ints, and also support double floating-point arithmetic.</p>

- Scalar Operations : Single Precision -- addss    %xmm0, %xmm1

<img width="2016" height="370" alt="QQ_1762508935478" src="https://github.com/user-attachments/assets/aa504882-61e1-4f2b-a3d0-61a73ec86890" />

- SIMD (Single Instruction Multiple Data) Operations : Single Precision -- addps    %xmm0, %xmm1

<img width="2010" height="366" alt="QQ_1762509007578" src="https://github.com/user-attachments/assets/ed9330ab-767f-4ecb-b26c-b65dbd0c74fc" />

- Scalar Operations : Double Precision -- addsd    %xmm0, %xmm1

<img width="1784" height="374" alt="QQ_1762509090278" src="https://github.com/user-attachments/assets/0ce0b2ee-74e4-4773-90e6-3787d27b6a38" />

<p>We will see if you write code to make use of these instructions you can really boost the performance of the compiler</p>

</br>

### FP Basics

</br>

- Arguments passed in %xmm0, %xmm1, ...

- Result returned in %xmm0

- All XMM registers caller-saved

```
float fadd(float x, float y)
{
	return x + y;
}
```

```
addss    %xmm1, %xmm0
ret
```

```
double fadd(double x, double y)
{
	return x + y;
}
```

```
addsd    %xmm1, %xmm0
ret
```

<p>FP memory referencing</p>

```
double dincr(double *p, double v)
{
	double x = *p;
	*p = x + v;
	return x;
}
```

```
# p in %rdi, v in %xmm0
movapd		%xmm0, %xmm1
movsd		(%rdi), %xmm0
addsd		%xmm0, %xmm1
movsd		%xmm1, (%rdi)
ret
```

<p>Other Aspects of FP Code</p>

- Lots of instructions

- Floating-point comparisions

  1. Instructions ucomiss and ucomisd
 
  2. Set conditional codes CF, ZF, and PF

- Using constant values : The SSE registers cannot be directly loaded with floating-point constants with immediate values.

  1. Set %xmm0 to 0 with instruction `xorpd		%xmm0, %xmm0`
 
  2. Others loaded from memory
 
</br>

# Chapter five -- Advanced-Topics

</br>

<p>We call it advanced topics but think of it more as miscellaneous topics.</p>

</br>

## Memory Layout

</br>

<p>What does the memory look like when you're running x86-64 programs.</p>

<p>x86-64 Linux Memory Layout : (now the machines limit you to actually only 47 bits of address)</p>

<p>Conceptually a memory is just a big array of bytes. And that's the view of the machine level programmer, even though that's not the actual implementation.</p>

<p>The whole part is called virtual memory, the organization looks very simple. But the underlying implementation is a complex management of various different memory types from disk memories to solid state disks, and to what's called DRAM which stands for dynamic RAM.</p>

- Stack

  1. Runtime stack (8MB limit)
 
  2. E.g. local variables
 
- Heap

  1. Dynamically allocated as needed
 
  2. When call malloc(), calloc(), new()
 
- Data

  1. Statically allocated data
 
  2. E.g. global vars, static vars, string constants
 
- Text / Shared libraries

  1. Excutable machine instructions
 
  2. Read-Only
 
<img width="354" height="1354" alt="QQ_1762524482143" src="https://github.com/user-attachments/assets/2de49a9e-fb56-4e49-b4ed-ca29fb1f5dcd" />

```
+-------------------------+  ← Top
| Stack                   |
|   ↓ decrease            |
+-------------------------+
| Heap                    |
|   ↑ increase            |
+-------------------------+
| Uninitialized Data (BSS)|
+-------------------------+
| Initialized Data (.data)|
+-------------------------+
| Read-Only Data (.rodata)|
+-------------------------+
| Text                    |
+-------------------------+  ← Bottom
```

<p>The stack size is limited to 8192 kb, and what that means is if you tried to access any memory via the stack pointer that was outside of the range of this 8 mb, you'd get a segmentation fault.</p>

<p>We call where the code (executable program) is sitting the text segment </p>

<p>And then the data, first of all there'll be a section for the data that's allocated at the program begins. So any global variables that you've declared will be in that section.</p>

<p>And then the heap is the part of memory that is allocated via call to malloc or one of its related functions. These variables varies dynamically as the program runs. It starts off with a small allocation and every time you call malloc and if you're not freeing memory, your memory requirements will keep growing. It will keep moving to larger addresses.</p>

<p>There is another place in memory for storing the code. The code that gets brought in that represents the library functions like printf and malloc. They are usually stored on the disk. And they get brought in when they get linked into your program when it firstly starts executing by a process known as <b>dynamic linking</b>.</p>

<p>Memory Allocation Example:</p>
 
```
char big_array[1L << 24]; 	// 16 MB
char huge_array[1L << 31]; 	// 2  GB

int global = 0;

int useless() {return 0;}

int main()
{
	void *p1, *p2, *p3, *p4;
	int local = 0;
	p1 = malloc(1L << 28);  // 256 MB
	p2 = malloc(1L << 8);   // 256 B
	p3 = malloc(1L << 32);	// 4   GB
	p4 = malloc(1L << 8);	// 256 B
}
```

<img width="1834" height="1428" alt="be02f9e2589dc2c7a02c62edc7f7ce25" src="https://github.com/user-attachments/assets/b44056e8-dfff-4142-8355-eeaf428fc081" />

<p>For one reason or another , it happens that the smaller chunks of memory allocations are allocated at address that are actually just a little bit above the pink section. The really big chunks of memory are allocated near the stack limit.</p>

<p>In general, what's happening is if you are to reference a memory address in this empty range, you'd get a segmentation fault.(Actually, the empty range doesn't have a corresponding physical memory mapping) If you keep allocating more with malloc, it will push the limits of what's addressable inward. In principle if you have got too much of a memory request, these two would hit each other and malloc would return zero.</p>

<hr>

<p>So why malloc allocations don’t all sit next to each other.</p>

<p>This is related to the underlying implementation strategy of malloc in glibc(GUN C Library). In C, glibc's malloc implementation chooses two different allocation methods depending on the requested size.</p>

| Allocation type | System Call | Typical Usage                                                            | Characteristics                                                          |
|-----------------|-------------|--------------------------------------------------------------------------|--------------------------------------------------------------------------|
| Samll Chunks    | brk/sbrk    | Usually for allocations smaller than a certain threshold (around 128 KB) | Allocates from a growing contiguous heap; Fast                           |
| Large Chunks    | mmap        | Used for large allocations (Exceeding the threshold)                     | Creates independent mappings; doesn't affect the main heap; easy to free |

<p>brk/sbrk</p>

- The brk system call adjusts the end of the heap segment

- All memory obtained this way forms one contiguous heap region

- When freeing memory, if the freed block isn't at the very top of the heap, it cannot be returned to the OS(It can only be reused internally)

```
Heap layout:
| stack | <-- high address
|       |
| heap  | <--- brk extends this upward
| data  |
| text  | <-- low address
```

<p>mmap</p>

- mmap asks the kernal to create separate virtual memory mappings, which can be freed individually.

- Suitable for large allocations, since it can release memory back to the OS immediately via munmap()

- These allocations are non-contiguous and independent of the main heap

<p>To summarize :</p>

```
[stack grows downward] <- High addresses
[unmapped gap]
[mmap region (large mallocs, shared libs, etc.)]
[unmapped gap]
[heap grows upward]
[text / data / bss]    <- Low addresses
```

<p>The operating system intentionally arranges memory like this : </p>

- The heap grows upward from low addresses

- The heap grows downward from high addresses

- The mmap region and empty gaps lie in between

<p>This design gives flexibility.</p>

- Prevent interference between the heap and stack

- Allow mmap to flexibly find available space between them

- Improve memory allocation flexibility(Some libraries, file mappings and anonymous mappings all rely on mmap)

</br>

## Buffer Overflow

</br>

<p>When you're running GDB it helps a lot when you're looking at these different addresses.</p>

<p>Recall: Memrory Referencing Bug</p>

```
typedef struct
{
	int a[2];
	double d;
} struct_t;

double fun(int i)
{
	volatile struct_t s;
	s.d = 3.14;
	s.a[i] = 1073741824; // Possibly out of bound
	return s.d;
}
```

```
fun(0) -> 3.14
fun(1) -> 3.14
fun(2) -> 3.1399998664856
fun(3) -> 2.00000061035156
fun(4) -> 3.14
fun(6) -> Segmentation fault
```

<p>Such problems are a big deal</p>

- Generally called a "buffer overflow"

  - When exceeding the memory size allocated for an array
 
- Why a big deal

  -  It's the #1 technical cause of security
 
     - #1 overall cause is social engineering / user ignorance
   
- Most common form

  1. Unchecked lengths on string inputs
 
  2. Particularly for bounded character arrays on the stack
 
     - Sometimes referred to as stack smashing

<p>local char buf[100] that gets overflowed: An classic stack smashing, <b>can overwrite the saved frame pointer or return address.</b></p>

<p>String Library code:</p>

- Implementation of Unix function gets()

  - No way to specify limit on number of characters to read

```
char *gets(char *dest)
{
	int c = getchar();
	char *p = dest;
	while (c != EOF && c != '\n')
	{
		*p++ = c;
		c = getchar();
	}
	*p = '\0';
	return dest;
}
```

- Similiar problems with other library functions

  1. strcpy, strcat: Copy strings of arbitrary length
 
  2. scanf, fscanf, sscanf, when given %s conversion specifier are used.

<p>Without knowing in advance how big that string is, it's possible that it will be too big for the buffer that's been allocated. One of the culprits is there's a whole class of library functions that don't perform any kinds of bounds checking. There's not an argument to the function that tells the function when it has to stop when it reach the limit.</p>

<p>Vulnerable Buffer Code</p>

```
void echo()
{
	char buf[4];
	gets(buf);
	puts(buf);
}

void call_echo()
{
	echo();
}
```



<p>Actually it can't handle more than three chars, bacause there should be room for the null character.</p>

<p>Call here a-nsp (Meaning no stack protector, we'll see what's nsp later)</p>

<p>Type a string : 012345678901234567890123. Output : 012345678901234567890123; Type a string : 0123456789012345678901234. It will hit a segmentation fault. (core dumped)</p>

<p>Observe it through assembly.</p>

```
00000000004006cf <echo>:
4006cf:    48 83 ec 18          sub    $0x18, %rsp
4006d3:    48 89 e7             mov    %rsp, %rdi
4006d6:    e8 a5 ff ff ff       callq  400680 <gets>
4006db:    48 89 e7             mov    %rsp, %rdi
4006de:    e8 3d fe ff ff       callq  400520 <puts@plt>
4006e3:    48 83 c4 18          add    $0x18, %rsp
4006e7:    c3                   retq
```

```
4006e8:    48 83 ec 08          sub    $0x8, %rsp
4006ec:    b8 00 00 00 00       mov    $0x0, %eax
4006f1:    e8 d9 ff ff ff       callq  4006cf <echo>
4006f6:    48 83 c4 08          add    $0x8, %rsp 		# return address
4006fa:    c3                   retq
```

<p>The first line is the code where you can tell how much memory get allocated for the buffer. 0x18 is 24 in decimal. A region of 24 bytes is allocated on the stack. And it's copying %rsp to %rdi. So gets is being called with a pointer to a buffer of size 24 </p>

</br>

### Buffer Overflow Stack

</br>

<img width="714" height="782" alt="QQ_1763218256276" src="https://github.com/user-attachments/assets/c5f94c0c-2bcc-421c-bbe9-7b2ee7ef35a1" />

<p>Let's see memory layout. The buffer normally big enough for four characters. There's 20 bytes unused.</p>

<p>So once I input beyond 23 charachers plus the null character, you'll see what I'm doing is corrupting the byte representation of the return address. So what happened for `0123456789012345678901234` here is rather than trying to return back to where call_echo called echo, it goes back to some other part of your code.(A segmentation fault occurs when program tries to access an address that is not allowed to access.)</p>

<p>For `012345678901234567890123` here I actually did overflow the buffer. The fact that iy didn't collapse was just a coincidence. "Returns" to unrelated code.</p>

<img width="1676" height="972" alt="QQ_1763274630603" src="https://github.com/user-attachments/assets/4c70be89-4dbb-43ea-a2ba-853108853999" />

</br>

### Code Injection Attacks

</br>

```
void P()
{
	Q();
	...
}

int Q()
{
	char buf[64];
	gets(buf);
	...
	return ...;
}
```

<img width="1108" height="1016" alt="847c3bd5d00bad1bd18c9d7ed75c35d3" src="https://github.com/user-attachments/assets/0d69b3dd-c62a-4bd2-9ea5-71027bbdd180" />

- Input string contains byte representation of excutable code

- Overwrite retrurn address A with address of buffer B

- When Q() excutes ret, will jump to exploit code

<p>A buffer overflow gives an attacker the opportunity to inject code into a program and execute it. The general idea is that there is a buffer which the attacker can fill with arbitrary bytes by supplying input to functions like gets or any other function that copies data without checking the length. The attacker places a sequence of bytes into this buffer that encodes executable machine instructions.</p>

<p>After writing the injected code into the buffer, the attacker may need to add some padding bytes whose actual values do not matter—in order to overwrite the return address correctly. The value B is chosen to be an address on the stack that corresponds to the beginning of this buffer, which is where the injected code resides.</p>

<p>When the function returns, it is supposed to jump back to its caller at address P. However, because the return address on the stack has been overwritten with B, the program counter will instead jump to B and begin executing the attacker’s injected instructions.</p>

<p>When you're trying to replace code, how do you make sure that your new number B provides the exactly address. It's pretty easy. For example, in previous one I could tell that it was allocating 24 bytes for the buffer, so I just make sure that the length of my exploit code plus the padiing is 24 bytes, then right after that comes the return address.</p>

<hr>

<p>What to do about buffer overflow attacks</p>

- Avoid overflow vulnerabilities

  - For example, use library routines that limit string lengths
 
    1. fgets instead of gets
   
    2. strncpy instead of strcpy
   
    3. Don't use scanf with %s conversion specification
   
       - Use fgets to read the string
      
       - or use %ns where n is a suitable integer

- Employ system-level protections

  - ASLR (Address Space Layout Randomization) : Randomized stack offsets : Stack repositioned each time program executes
 
    1. At start of program, allocate random amount of space on stack
   
    2. Shifts stack addresses for entire program
   
    3. Makes it difficult for hacker to predict beginning of inserted code
   
    4. All the local storage on the stack will shift up and down from one run to another(The allocation by malloc also has randomness)
   
  - Nonexecutable code segments
 
    1. In traditional x86, can mark region of memory as either "read-only" or "writeable" : Can execute anything readable
   
    2. X86-64 added explicit "execute" permission
   
    3. Stack marked as non-executable. Any attempt to execute this code will fail

- Have compiler use "stack canaries"

  - Idea
 
    1. Place special value ("canary") on stack just beyond buffer
   
    2. Check for corruption before exiting function
   
  - GCC Implementation
 
    1. -fstack-protector
   
    2. Now the default(disabled earlier)

<img width="1108" height="470" alt="QQ_1763359388835" src="https://github.com/user-attachments/assets/c3971d1d-19cf-42e8-a963-67b6e8e9d121" />

<p>Lets see what that canary code looks like:</p>

```
echo:
40072f:    sub     $0x18, %rsp
400733:    mov     %fs:0x28, %rax					/
40073c:    mov     %rax, 0x8(%rsp)					/
400741:    xor     %eax, %eax
400743:    mov     %rsp, %rdi
400746:    callq   4006e0 <gets>
40074b:    mov     %rsp, %rdi
40074e:    callq   400570 <puts@plt>
400753:    mov     0x8(%rsp), %rax					/
400758:    xor     %fs:0x28, %rax					/
400761:    je      400768 <echo+0x39>
400763:    callq   400580 <__stack_chk_fail@plt>	/
400768:    add     $0x18, %rsp
40076c:    retq
```

<p>%fs:0x28 is the address of the stack canary. On 64-bit Linux, %fs is a segment register used to access TLS(Thread Local Storage). Inside TLS, there is a region reserved by glibc, which stores various thread-local data:</p>

- The stack canary(Stack protector value)

- errno

- pthread-related value

- TLS module pointers

`FS segment base + offset 0x28 = the address of the stack canary`

<img width="580" height="942" alt="QQ_1763363453001" src="https://github.com/user-attachments/assets/39581cce-6520-41d5-aba3-c1ea544c6e14" />

<p>At offset 8 from the stack pointer it's putting a value in 8 bytes, storing it as canary value</p>

<hr>

<p><b>Return-Oriented Programming Attacks</b></p>

- Challenge for hacker

  1. Stack randomization makes it hard to predict buffer location
 
  2. Marking stack nonexecutable makes it hard to insert binary code
 
- Alternative Strategy

  1. Use existing code
 
     - E.g., library code from stdlib
    
  2. String together fragments to achieve overall desired outcome
 
  3. Doesnot overcome stack canaries
 
- Construct program from gadgets

  1. Sequence of instructions ending in ret : Encoded by single byte 0x3c
 
  2. Code positions fixed from run to run
 
  3. Code is executable

<p>Stack canaries are extremely effective, and unless an attacker can leak their value, there’s essentially no way to bypass them. But the other two protections—stack randomization and a non-executable stack—can still be worked around.</p>

<p>Stack randomization only changes where the stack is located; it does not randomize the locations of global variables or the program’s code segment. This means that even though the attacker cannot reliably find their buffer on the stack, the code segment of the program or shared libraries still remains at predictable or discoverable addresses.</p>

<p>Since the attacker cannot inject and execute their own code because of the non-executable stack, a different strategy is used: reusing existing code. If the attacker can locate the program’s code or library code, they can chain together many small instruction sequences—called gadgets—each ending with a ret. By arranging these gadget addresses on the stack, the attacker can construct a meaningful sequence of operations using only existing instructions, achieving arbitrary behavior without injecting new code.</p>

<p>Example #1 : Use tail end of existing functions</p>

```
long ab_plus_c(long a, long b, long c)
{
	return a * b + c;
}
```

```
00000000004004d0 <ab_plus_c>:
4004d0:   48 0f af fe          imul   %rsi, %rdi
4004d4:   48 8d 04 17          lea    (%rdi,%rdx,1), %rax
4004d8:   c3                   retq
```

<p>Gadget address : 0x4004d4</p>

<p>Example #2 : Repurpose byte codes</p>

```
void setval(unsigned *p)
{
	*p = 3347663060u;
}
```

```
<setval>:
4004d9:		c7 07 d4 48 89 c7	movl $c78948d4, (%rdi)
4004df:		c3					retq
```

<p><b>48 89 c7 encodes : movq	%rax, %rdi</b></p>

<p>Gadget address : 0x4004dc</p>

<img width="1300" height="564" alt="QQ_1763369923592" src="https://github.com/user-attachments/assets/922d9cdb-39b0-44f1-8392-26fe513bf08d" />

- Trigger with ret instruction

  - Will start executing Gadget 1
 
- Final ret in each gadget will start next one

<p>Imagine I fill up my buffer instead of with executable code, I could fill it up with a series of gadget addresses.</p>

<p>It's using the peculiar behavior of how return works in x86.</p>

</br>

## Union

</br>

<p>Union Allocation</p>

- Allocate according to larget element

- Can only use one field at a time

<p>Use union to access bit pattern : When you take a unsigned value and you cast it to a float, you actually changed the bits. But union doesn't change the bit.</p>



































































































































































































































































































































































































































































































































































