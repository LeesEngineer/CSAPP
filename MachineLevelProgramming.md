# Chapter One Basics

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


















































































































































































































































































































































































































































































































































































































































































