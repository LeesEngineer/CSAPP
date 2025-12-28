<img width="1620" height="1118" alt="QQ_1766736629092" src="https://github.com/user-attachments/assets/504e4542-e683-4fba-8ae3-5d20c17ce7b8" /></br>

# Linking

</br>

```
// sum.c
int sum(int *a, int n)
{
    int i, s = 0;

    for(i = 0; i < n; i ++)
        s += a[i];
    return s;
}
```

```
// main.c
int sum(int *a, int n);

int array[2] = {1, 2};

int main()
{
    int val = sum(array, 2);
    return val;
}
```

<p>That's kind of an odd thing to return your exit status like that, we just did it so that the compiler wouldn't optimize away all of the code. Because compiler must ensure that the return value val is calculated correctly, so it can't arbitrarily delete the sum() function or array-related code.</p>

<p>Let's look at what happens when we want to compile those two modules.</p>

</br>

## Static Linking

</br>

<p>Programs are translated and linked using a compiler driver: </p>

```
linux> gcc -Og -o prog main.c sum.c
linux> ./prog
```

<img width="1016" height="874" alt="QQ_1766498221056" src="https://github.com/user-attachments/assets/f62e417b-3990-450f-8972-59e23c0e2440" />

<p>The GCC calls a series of translators on these modules, first calls the C preprocessor(cpp), then calls the compiler(cc1). What the compiler generates is translated by the assembler(as).</p>

<p>These .o files are called relocatable object files. Then smash them together.</p>

<hr>

<p>So why do we allow this so-called separate compilation?</p>

- Reason 1: Modularity

  - Program can be written as a collection of smaller source file, rather than one monolithic mass.
 
  - Can build libraries of common functions (more on this later). e.g., Math library, standard C library
 
- Reason 2: Efficiency

  - Separate compilation (Time): Change one source file, compile, and then relink. No need to recompile other source file.
 
  - Libraries (Space): Common functions can be aggregated into a single file. Yet executable files and running memory images contain only code for the functions they actually use.

</br>

## What Do Linkers Do

</br>

- Step 1: Symbol resolution

  - Programs define and reference symbols (<b>global variables</b> and functions):
 
    - void swap() {} /* define symbol swap */
   
    - swap(); /* reference symbol swap */
   
    - int *xp = &x; /* define symbol xp, reference x */
 
  - Symbol definitions are stored in object file (by assembler) in symbol table.
 
    - Symbol table is an array of structs
   
    - Each entry includes name, size, and location of symbol
   
  - During symbol resolution step, the linker associates each symbol reference with exactly one symbol definition
 
- Step 2: Relocation

  - Merges separate code and data sections into single sections that can be directly loaded and executed on the system.
 
  - Relocates symbols from their relative locations in the .o files to their final absolute memory locations in the executable.
 
  - Updates all references to these symbols to reflect their new positions
 
<p>When it does this merging, it has to figure out where each symbol is going to be stored. Because initially functions are stored at some offset in object module. Linker doesn't where these functions are actually going to be loaded into memory. So before relocation the address of a function in the module is just its offset in the module and similarly for data.</p>

<p>During relocation the linker decides on where each symbol is going to be ultimately located in memory when the program executes. It binds those absolute memory locations to the symbols.</p>

<p><b>For each definition figure out where it's going to go and for each reference then update that reference.</b> So it now points to the right spot.</p>

</br>

## Symbol resolution

</br>

### Object Files (Modules)

</br>

<p>There're three kinds of object files (Modules)</p>

- Relocatable object file (.o file)

  - Contains code and data in a form that can be combined with other relocatable object files to form executable object file.
 
  - Each .o file is produced from exactly one source (.c) file. This is the output of the assembler
 
- Executable object file (a.out file)

  - Contains code and data in a form that can be copied directly into memory and then executed
 
- Shared object file (.so file)

  - Special type of relocatable object file that can be loaded into memory and linked dynamically, at either load time or run-time.
 
  - Called Dynamic Link Libraries (DLLs) by Windows
 
<p>Shared object file is a modern technique for creating shared libraries.</p>

</br>

#### Executable and Linkable Format (ELF)

</br>

<p>Now object modules come in a standard format called ELF. <b>It's the standard binary format for object files.</b> it's a unified format for .o file, a.out file and .so file.</p>

- ELF header

  - Word size, byte ordering, file type (.o, exec, .so), machine type, etc.
 
- Segment header table (required for executables)

  - Page size, virtual addresses memory segments (sections), segment sizes.
 
- .text section

  - Code
 
- .rodata section

  - Read-only data: jump tables, ...
 
- .data section

  - Initialized global variables

- .bss section

  - Uninitialized global variables
  
  - A better way to remember what it means is "Better Save Space"
 
  - Has section header but occupies no space
 
- .symtab section

  - Symbol table
 
  - Procedure and static variable names
 
  - Section names and locations
 
- .rel.text section

  - Relocation info for .text section
 
  - Addresses of instructions that will need to be modified in the executable
 
  - Instructions for modifying
 
- .rel.data section

  - Relocation info for .data section
 
  - Addresses of pointer data that will need to be modified in the merged executable
 
- .debug section

  - Info for symbolic debugging (gcc -g)
 
- Section header table

  - Offsets and sizes of each section

<img width="576" height="894" alt="QQ_1766561711866" src="https://github.com/user-attachments/assets/aaa64ed1-761d-40e1-84fe-f422b7ae0a2c" />

<p>They are very structured. They're broken up into sections. At the beginning is a header that defines some <b>general informations about this binary</b> like the word size and byte ordering. </p>

<p>And then there's what's so called segment header table. <b>It's only defined for the executable object files.</b> And it indicates the location of all the different segments of the code in memory, so where is your stack, where do your shared libraries go and where is your initialized and uninitialized data. All these various sections are defined in the segment header table.</p>

<p>And then there's the code itself which is called the .text section for sort of arcane historical reasons. Then that's followed by read-only data, such as the jump tables in switch statements. So the .text and .rodata have the property that they're both read-only (you don't write to them)</p>

<p>Then that's followed by the .data section which contains space for all of your initialized global variables.</p>

<p>Then there's a section called .bss which defines the uninitialized global variables. It doesn't actually take up any space and it only records the size, because they're uninitialized droids. <b>But there are entries in the symbol table for them.</b> And <b>when this program gets loaded these variables are going to need, they have to have space allocated for them.</b></p>

<p>So if you have a separate section for the uninitialized variable, they don't have to consume any room in the .o file.</p>

<p>Ok, there's also a section for the symbol table. It's an array of struct for procedures, global variables and <b>anything defined with the static attribute</b>. Each one of these symbols gets an entry in the symbol table</p>

<p>Then there're two sections that contain relocation info. When the linker identifies all the references to symbols, it will put a <b>note</b> so that I have to remember to fix the reference to this symbol up when I actually create the executable. </p>

<p>So a relocation entry is like a note. Assembler makes the linker to say you have to fix up this reference. Because I don't know where this symbol is actually going to be stored in memory when it's loaded.</p>

<p>Okey then there's a debug section that contains information that relates line numbers in the source code to line numbers in the machine code. You get it when you compile with -g.</p>

<p>Then there's a header table that tells you where all these different sections start.</p>

</br>

### Linker Symbols

</br>

<p>To a linker there's three different kinds of symbols.</p>

- Global symbols

  - Symbols defined by module m that can be referenced by other modules
 
  - E.g., non-static C functions and non-static global variables
 
- External symbols

  - Global symbols that are referenced by module m but defined by some other module.
 
- Local symbols

  - Symbols that are defined and reference exclusively by module m
 
  - E.g., C functions and global variables defined with static attribute
 
  - <b>Local linker symbols are not local program variables</b>

<p>If we have a program that consists of multiple modules, and we can compile each one of those modules into a .o file and will call functions that aren't defined but defined by other modules. The compiler doesn't throw an error, because it's assuming that those are defined in other modules and it assumes the linker will be able to find them and determine the addresses. </p>

<b>Local non-static C variables vs. local static C variables</b>

- Local non-static C variables: stored on the stack

- Local static C variables: stored in either .data, or .bss

```
int f()
{
    static int x = 0;
    return x;
}
int g()
{
    static int x = 1;
    return x;
}
```

<p>Because it's local, its scope is limited to this function. So this variable x can only be referenced within f. But because it's declared with static attribute, it's not stored on the stack. <b>Compiler will allocate space in .data for each definition of x</b>.</p>

<p>But compiler will creates local symbols in the symbol table with unique names, e.g., x.1 and x.2. They will also get symbol table entries just like any other symbol.</p>

<hr>

<p>How linker resolves duplicate symbol definitions?</p>

<p>As I said the linker associates each symbol reference to exactly one unique symbol definition. How does it do If there're multiple symbol definitions across all the modules. To understand this we need to define symbols as being either strong or weak.</p>

<p>Program symbols are either strong or weak: </p>

- Strong: procedures and initialized globals

- Weak: uninitialized globals

<p>Linker's Symbol Rules: </p>

- Rule 1: Multiple strong symbols are not allowed

  - Each item can be defined only once 
 
  - Otherwise: Link error
 
<p>That means if we declare several functions with the same name across multiple modules, linker will throw an error. (<b>But CPP supports overloading</b>) </p>

- Rule 2: Given a strong symbol and multiple weak symbols, choose the strong symbol

  - References to the weak symbol resolve to the strong symbol
 
  - It will associate all references of the symbol to that strong symbol

- Rule 3: If there are multiple weak symbols, pick an arbitrary one

  - Can override this with gcc -fno-common
 
<p>If you compile your program with this no common argument. Then multiple weak symbols will cause a linker error.</p>

<p>Linker error is the hardest to debug</p>

<p>Now we have two modules.</p>

```
// Module one
int x;
p1() {}

// Module two
int x;
p1() {}
```

<p>There're two strong symbols, so that's an error. But we've got two weak symbols. If these modules are referencing x the linker will just <b>pick one of these to serve as the definition</b>. </p>

```
// Module one
int x;
int y;
p1() {}

// Module two
double x;
p2() {}
```

<p>But if we declare these weak symbols with different types, we will get into trouble. Writes to x in p2 might overwrite y!</p>

<p>When multiple modules contain weak symbols with the same name, linker's handling principle is to allocate a block of memory equals to "maximum size + maximum alignment requirement" for these symbols.</p>

<p>X was defined across these two modules with different types. x needs 4 byte in module one, and needs 8 byte in module two. <b>When linker sees these two weak symbols, it allocates the larger size block of contiguous memory.</b> All of the references to x will point to this block of memory. If we reference x in module one it'll be an 8-bye right, so it will overwrite y.</p>

<p>From module one's perspective, the memory layout is as followed: `| x (4B) | y (4B) |`. But x is 8-byte and 8-aligned to module two.</p>

<p>Evil!</p>

```
// Module one
int x = 7;
int y = 5;
p1() {}

// Module two
double x;
p2() {}
```

<p>Linker will always associate all references to x to this integer sized symbol. But writes to x in p2 will also overwrite y!</p>

<p>Before linking, x is eight-byte in the code compiled fron module two. There's a strong symbol of x in module one and all the references will be associated to it, but their size and alignment will still be taken into consideration. <b>The linker doesn't check the type and only guarantees that there's enough space for it.</b></p>

<p>Nasty!</p>

<p>Nightmare scenario: two identical weak structs, compiled by different compilers with different alignment relus.</p>

<p>Because we're following a standard ABI, we can compile our code with multiple compilers.</p>

<p>Avoid global variables if you can, otherwise: </p>

- Use static if you can

- Initialize if you define a global variable

- Use extern if you reference an external global variable

<p>There's more to say about 'extern':</p>

<p>The purpose of 'extern' is not to "define a variable", but rather to declare a global symbol that is not defined in the current file. It tells the compiler: This name exists, but don't allocate memory for it and don't generate weak symbol.</p>

```
// a.c
int x = 10;

// b.c
int x;
```

<p>This isn't a refernce to x in b.c, but rather a new x is defined which is a weak symbol. The linker will merge them. And if an error occurs, you'll starting debugging hellish bug.</p>

```
// globals.h
extern int x;

// b.c
#include "globals.h"
```

</br>

## Relocation

</br>

<p>Now at this point, the linker has associated every symbol references with their definition. Now it has to take these relocatable object files and smush them together to create an big executable.</p>

<img width="626" height="686" alt="QQ_1766652194133" src="https://github.com/user-attachments/assets/65c26b23-d30b-4863-92e8-4633542d3ce0" />

<p>There's their system code that actually runs before and after you program. When your program run it starts executing a setup code from lib.c that initializes things and the last thing is it calls main and passes rc and rv.</p>

<p>When the linker relocates these object files, it takes all of the code the text sections from each of the modules, and puts them together contiguously in the .text section for the executable object file. It does the same thing with .data. It also emerges the symbol tables and the debug information as well.</p>

<img width="758" height="736" alt="QQ_1766652658091" src="https://github.com/user-attachments/assets/b0991cee-4615-4a29-8a24-7b287f9d6db7" />

<p>Modules require linker to figure out where it's going to store these symbols when program gets loaded. It has to pick an address for main so that function will start at some absolute address.</p>

</br>

### Relocation Entries

</br>

<p>The problem is that when the code gets compiled, the compiler doesn't know what addresses the linker is going to pick. So the compiler creates reminders to linker called relocation entries which are then stored in the relocation sections of the object file.</p>

```
int array[2] = {1, 2};

int main()
{
    int val = sum(array, 2);
    return val;
}
```

```
0000000000000000 <main>:
   0:   48 83 ec 08             sub    $0x8, %rsp
   4:   be 02 00 00 00          mov    $0x2, %esi
   9:   bf 00 00 00 00          mov    $0x0, %edi
                        a: R_x86_64_32 array

   e:   e8 00 00 00 00          callq  13 <main+0x13>
                        f: R_x86_64_PC32 sum-0x4
  13:   48 83 c4 08             add    $0x8, %rsp
  17:   c3                      retq
```

<p>`0:   48 83 ec 08             sub    $0x8, %rsp`: This is standard practice at the beginning of a function: allocate space on the stack for local variables.</p>

<p>These relocation entries are <b>instructions</b> to the linker that there's a reference to a symbol that's going to have to be patched up when the code is relocated and merged into the executable.</p>

<p>The compiler creates two relocation entries. The first one for the reference to array a. We're moving the address of the array into %edi for the first argument. But the compiler doesn't know what the address is going to be. It just moves in an immediate value of 0 into %edi temporarily. 'bf' is mov instruction and then there're all zeros. <b>Then it places this relocation entry in the relocation section of main.c</b>. As followed like this: </p>

```
offset = a
type   = R_x86_64_32
symbol = array
```

<p>The function starts at offsets zero in the .text section of the module. If there were other functions in this module, they would follow immediately after. So you can see compiler is just generating offsets of these instructions from the beggining of the .text section.</p>

<p>It includes this relocation entry which tell the linker that when you're relocating main.o add offset a in this .text section. You've got a reference to an array a in the form of a 32-bit address. Eventually the linker has to patch up the four bytes starting at address a with the absolute address of the symbol. </p>

<p>And similarly we have reference to sum. Compiler knows nothing about sum. In this case, it does a call with all zeros. The relocation entry tells linker that at offset f you've got a <b>four-byte pc-relative reference</b> to call sum.</p>

<p>There's an option to include a bias in the offset, since calls are always resolved using pc-relative addressing. The value is going to be placed at these four bytes that offset f. <b>It's going to be an offset from the current %rip value</b>. The program counter always points to the next instruction.</p>

<p>The point is that there's enough information for the linker to fill in the right address.</p>

<p>Now look at the relocated .text section. We compile this code into an executable and then we use objdump to disassemble it.</p>

```
00000000004004d0 <main>:
   4004d0:      48 83 ec 08          sub    $0x8,%rsp
   4004d4:      be 02 00 00 00       mov    $0x2,%esi
   4004d9:      bf 18 10 60 00       mov    $0x601018,%edi    # %edi = &array
   4004de:      e8 05 00 00 00       callq  4004e8 <sum>      # sum()
   4004e3:      48 83 c4 08          add    $0x8,%rsp
   4004e7:      c3                   retq

00000000004004e8 <sum>:
   4004e8:      b8 00 00 00 00       mov    $0x0,%eax
   4004ed:      ba 00 00 00 00       mov    $0x0,%edx
   4004f2:      eb 09                jmp    4004fd <sum+0x15>
   4004f4:      48 63 ca             movslq %edx,%rcx
   4004f7:      03 04 8f             add    (%rdi,%rcx,4),%eax
   4004fa:      83 c2 01             add    $0x1,%edx
   4004fd:      39 f2                cmp    %esi,%edx
   4004ff:      7c f3                jl     4004f4 <sum+0xc>
   400501:      f3 c3                repz retq
```

<p>Using PC-relative addressing for sum(): <b>0x4004e8 = 0x4004e3 + 0x5</b></p>

<p>At address 1004df, those four bytes which were original zero have been updated with the actual address of array in memory at runtime. The linker decided that the array is going to go at address 0x601018.</p>

<p>The call to sum is also been updated. In this case, the function that you want to call is at 0x4004e3 + 5. The address is been updated with the pc-relative address of five. When it computes the absolute address of sum, it will take the current value of program counter which points to the next instruction so 0x4004e3, and it will add this value to the pc-relative address in the immediate field which is a two's complement integer so you can go + or -.</p>

<p>Compiler is smart. It computed the relocation entry. The linker is just blindly going through each of these entries and just doing what it's telling. The result is that all of these references have been patched up with absolute addresses</p>

</br>

## Loading Executable Object Files

</br>

<p>Once the linker creates an object file, the object file can be loaded directly into memory with no further modification.</p>

<p>If you look at all of the read-only sections. All the code is in the .text and <b>jump tables are in .rodata</b>. All of this can be loaded directly into memory so that it forms the so-called a read-only segment.</p>

<img width="1620" height="1118" alt="QQ_1766736629092" src="https://github.com/user-attachments/assets/0639e0e7-14b0-43b4-aa28-06eb218da2de" />

<p>This is the address space that every Linux program sees. The memory above stack is restricted to the kernal. There's a regoin in the huge gap between stack and heap. .so file all get loaded into this memory mapped region for shared libraries.</p>

<p>Small blocks of memory are allocated by brk and stored contiguously in the heap. Large blocks of memory (e.g., >128KB) are allocated directly by mmap. The top of the heap is indicated by this global variable 'brk' which is maintained by the kernal. </p>

</br>

## Packaging Commonly Used Functions

</br>

<p>One of the advantages of linking is that allows us to create libraries. </p>

<p>Awkward, given the linker framework so far: </p>

- Option 1: Put all functions into a single source file

  - Programmers link big object file into their programs
 
  - Space and time inefficient
 
- Option 2: Put each function in a separate source file

  - Programmers explicitly link appropriate binaries into program
 
  - More efficient, but burdensome on programmer
 
<p>The developers of Unix come up with the first solution called static libraries. This is the old solution.</p>

- Concatenate related relocated object files into a single file with an index (called an <b>archive</b>).

- Enhance linker so that it tries to resolve unresolved external references by looking for the symbols in one or more archives

- If an archive member file resolves reference, link it into the executable

<p>You create this archive called a .a file which is a collection of .o file where each .o file contains a function. You use option 2 to create a bunch of .o files, then you use a program called an <b>archiver or AR</b> to take these .o file and put them together in a big file called an archive <b>with a table of contents that tells you the offset of each of these .o files.</b> Then you pass the archive to the linker, and linker only takes the .o files that are actually referenced and links them into the code.</p>

<img width="1566" height="654" alt="QQ_1766743558918" src="https://github.com/user-attachments/assets/5ed75489-96ce-4ea0-b3df-8e9c8536ed9d" />

- Archiver allows incremental updates

- Recompile function that changes and replace .o file in archive (We can recreate that archive at anytime)

<hr>

<p>Commonly Used Libraries: </p>

- libc.a (the C standard library)

  - 4.6 MB archive of 1496 object files
 
- libm.a (the C math library)

  - 2 MB archive of 444 object files
 
  - floating point math (sin, cos, tan, log, exp, sqrt, ...)
 
<p>Use `ar -t libc.a | sort` to show all of the object files</p>

</br>

### Linking With Static Libraries

</br>

<p>Create a little example here: </p>

```
// main2.c
#include <stdio.h>
#include "vector.h"

int x[2] = {1, 2};
int y[2] = {3, 4};
int z[2];

int main()
{
    addvec = (x, y, z, 2);
    printf("z = [%d, %d]\n", z[0], z[1]);
    return 0;
}
```

```
// libvector.a

// addvec.c
void addvec(int *x, int *y, int *z, int n)
{
    int i;
    for(i = 0; i < n; i ++)
    [
        z[i] = x[i] + y[i];
    }
}

// multvec.c
void multvec(int *x, int *y, int *z, int n)
{
    int i;
    for(i = 0; i < n; i ++)
    [
        z[i] = x[i] * y[i];
    }
}
```

<img width="1632" height="984" alt="QQ_1766750247293" src="https://github.com/user-attachments/assets/59c7af06-bd27-4244-906c-5143601db330" />

<p>We're doing this linke at compile time.</p>

- Linker's algorithm for resolving external references:

  - Scan .o files and .a files in the command line order
 
  - During the scan, keep a list of the current unresolved references
 
  - As each new .o or .a file, obj, is encountered, try to resolve each unresolved reference in the list against the symbols defined in obj
 
  - If any entries in the unresolved list at end of scan, then error.

- Problem:

  - Command line order matters
 
  - Moral: put libraries at the end of the command line.
 
```
unix> gcc -L. libtest.o -lmine # libtest calls function that's declared in lmine.a
unix> gcc -L. -lmine libtest.o
libtest.o: In function `main':
libtest.o(.text+0x4): undefined reference to `libfun'
```

<p>The key is that the linker will try to <b>resolve these references from left to right on the command line</b>. The order that you put your files on the command line makes a difference. You can get sort of weird linker errors if you use the wrong order.</p>

</br>

### Modern Solution: Shared Libraries

</br>

- Static libraries have the following disadvantages:

  - Duplication in the stored executables (every function needs libc)
 
  - Duplication in the running executables
 
  - Minor bug fixes of system libraries require each application to explicitly relink

<p>The modern solution is to use dynamic libraries or shared libraries. Shared libraries provide a mechanism to ensure that <b>there is only a single instance of a function like printf, and every program running on the system will share that copy.</b> </p>

- Object files that contain code and data that are loaded and linked into an application dynamically, at either load-time or run-time

- Also called: dynamic linke libraries, DLLs (Windows), .so files

<hr>

- Dynamic linking can occur when executable is first loaded and run (<b>load-time linking</b>).

  - Common case for Linux, handled automatically by the dynamic linker (ld-linux.so)

  - Standard C library (libc.so) usually dynamically linked

- Dynamic linking can also occur after program has begun (<b>run-time linking</b>)

  - In Linux, this is done by calls to the dlopen() interface
 
    - Runtime library interpositioning
   
- Shared library routines can be shared by multiple processes

<img width="1684" height="1082" alt="QQ_1766839637784" src="https://github.com/user-attachments/assets/e2fe46ee-5741-43e5-a317-7219c2e59e3c" />

<p>Instead of creating an archive we create a .so file using the shared argument to GCC.</p>

<p>At linking time, the linker doesn't actually copy functions and it <b>pushes in a relocation entry</b> to say that those references to functions will need to be resolved when program is loaded. So it's partially linked but it's not fully linked. </p>

<p><b>Then the loader calls the dynamic linker</b> which takes these .so file and then resovles all the references to any unresolved reference</p>

<p><b>The address of aadvec and printf aren't determined by dynamic linker until the program is loaded.</b> So that dynamic linker does it goes through a similar process that static linker did.(sort of fixing up references) </p>

</br>

### Dynamic Linking at Run-time

</br>

- Regular dynamic linking: All dependent .so files are loaded as soon as the program starts.

  - <b>Dependencies are hardcoded in the ELF file</b>
 
  - Loaded as soon as the program starts
 
  - Controlled by the linker and ld.so file

- dlopen: Libraries are loaded only when the program reaches a specific line of code.

  - Dependencies are not written into the ELF file.
 
  - They are loaded only at this point in the runtime.
 
  - Controlled by the programmer.

```
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>

int x[2] = {1, 2};
int y[2] = {3, 4};
int z[2];

int main()
{
    void *handle;
    void (*addvec)(int *, int *, int *, int);
    char *error;

    /* Dynamically load the shared library that contains addvec() */
    handle = dlopen("./libvector.so", RTLD_LAZY);
    if(!handle)
    {
        fprintf(stderr, "%s\n", dlerror());
        exit(1);
    }

    /* Get a pointer to the function we just loaded */
    addvec = dlsym(handle, "addvec");
    if((error = dlerror()) != NULL)
    {
        fprintf(stderr, "%s\n", dlerror());
        exit(1);
    }

    /* Now we can call addvec() */
    addvec(x, y, z, 2);
    printf("z = [%d, %d]\n", z[0], z[1]);

    /* Unload the shared library */
    if(dlclose(handle) < 0)
    {
        fprintf(stderr, "%s\n", dlerror());
        exit(1);
    }
    return 0;
}
```

<p>You can arbitrarily decide to load link and call a function from a shared .so file. Because <b>there's an interface called dlopen</b>.</p>

<p>dlopen is a runtime dynamic linking interface for Unix/Linux. In short, dlopen allows programs to manually load .so dynamic libraries at runtime and access their functions and variables.</p>

- RTLD_NOW

  - Immediately resolve all external symbols and complete all relocations.
 
  - If a symbol is missing or a dependency library is missing, dlopen will fail.
 
- RTLD_LAZY

  - Only non-function symbols are processed.
 
  - Function symbols are not resolved immediately.
 
  - On the first function call:Resolution is triggered only then.

<p>dlopen returns a handle that you use in subsequent calls. If the handle is null there was some kind of errors like this .so file doesn't exist.</p>

<p>`dlclose` reduces the library's reference count. When the reference count reaches 0, the .so file will be unloaded (munmapped), and its code segment, GOT, and PLT all disappear. Your function pointers are pointing to memory that already is unloaded.</p>

<p>So linking can happen at different times in a program's lifetime: </p>

- Compile time (when a program is compiled)

- Load time (when a program is loaded into memory)

- Run time (while a program is executing)

</br> 

# Library interpositioning

</br>

<p>This technique allows you to use linking to actually intercept on function calls in libraries like the standard C library. It's all enabled by linking.</p>

<p>Interpositioning can occur at compile-time, link-time and run-time.</p>

<p>We intercept a function call for some reason maybe record some statistics or do some error checking (Security, Debugging, Monitoring and Profiling), and then call the real function as intended. The idea is we're going to create wrappers, and when the program calls a function we're going to execute its wrapper.</p>

<p><b>The symbol resolution rule for dynamic linking: the symbol found first is used first.</b></p>

<p>Without modifying the program's source code, you can use your own implementation to "intercept/replace" library function calls. This is a dynamic linking technique; essentially, it makes the dynamic linker see "your function" first, instead of the original library's function.</p>

<b>Monitoring and Profiling: </b>

- Count number of calls to functions

- Characterize call sites and arguments to functions

- Malloc tracing

  - Detecting memory leaks
 
  - <b>Generating address traces</b>
 
<p>In malloc lab, you're going to evaluate you malloc using a trace which is generated from this technique.</p>

```
#include <stdio.h>
#include <malloc.h>

int main()
{
    int *p = malloc(32);
    free(p);
    return (0);
}
```

- Goal: trace the addresses and sizes of the allocated and freed blocks without breaking the program, and without modifying the source code

- Thee solutions: interpose on the lib malloc and free functions at compile time, link time, and run time.

<b>Compile-time Interpositioning</b>

```
#ifdef COMPILETIME
#include <stdio.h>
#include <malloc.h>

/* malloc wrapper function */
void *mymalloc(size_t size)
{
    void *ptr = malloc(size);
    printf("malloc(%d) = %p\n", (int) size, ptr);
    return ptr;
}
void myfree(void *ptr)
{
    free(ptr);
    printf("free(%p)\n", ptr);
}
#endif
```

<p>Here is the trick in malloc.h</p>

```
#define malloc(size) mymalloc(size)
#define free(ptr) myfree(ptr)

void *mymalloc(size_t size);
void free(void *ptr);
```

```
linux> make intc
gcc -Wall -DCOMPILETIME -c mymalloc.c
gcc -Wall -I. -o intc int.c mymalloc.o
linux> make runc
./intc
malloc(32) = 0x1edc010
free(0x1edc010)
linux>
```

<b>Link-time Interpositioning</b>

```
#ifdef LINKTIME
#include <stdio.h>

void *__real_malloc(size_t size);
void *__real_free(void *ptr);

void *__wrap_malloc(size_t size)
{
    void *ptr = __real_malloc(size); /* Call libc malloc */
    printf("malloc(%d) = %p\n", (int)size, ptr);
    return ptr;    
}
void __wrap_free(void *ptr)
{
    __real_free(ptr);
    printf("free(%p)\n", ptr);
}
```

<p>In the first solution, we have to compile it. But we can use link-time solution to avoid that compilation. At link time, we do the interpositioning by calling the linker with this -Wl argument.</p>

```
linux> make intl
gcc -Wall -DLINKTIME -c mymalloc.c
gcc -Wall -c int.c
gcc -Wall -Wl, --wrap, malloc -Wl, --wrap, free -o intl
int.o mymalloc.o
linux> make runl
./intl
malloc(32) = 0x1aa0010
free(0x1aa0010)
linux>
```

- The "-Wl" flag passes argument to linker, replacing each comma with a space

- The "--wrap, malloc" arg instructs linker to resolve references in a special way:

  - Refs to malloc should be resolved as __wrap_malloc
 
  - Refs to __real_malloc should be resolved as malloc
 
<b>Run-time  Interpositioning</b>

```
#ifdef RUNTIME
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>

void *malloc(size_t size)
{
    void *(*mallocp)(size_t size);
    char *error;

    mallocp = dlsym(RTLD_NEXT, "malloc"); /* Get addr of libc malloc */
    if((error = dlerror()) != NULL)
    {
        fputs(error, stderr);
        exit(1);
    }
    char *ptr = mallocp(size);
    printf("malloc(%d) = %p\n", (int)size, ptr);
    return ptr;
}
```

<p>You can also do interpositioning at run-time. So you don't even need access to the .o files <b>all you need is access to the executable.</b> And for every program we have access to the executable. <b>So think about we can take any program.</b> We can interpose on its library calls at runtime.</p>

<p>`dlsym(RTLD_NEXT, "malloc")` tells dynamic linker that <b>after the current library containing malloc, find the next implementation called malloc (that is, the malloc in libc).</b> Because it is an alternative implementation of malloc, <b>it's resolved before libc through library interpositioning.</b></p>

<p>Then the interpositioning happens when the program is loaded.</p>

```
linux> make intr
gcc -Wall -DRUNTIME
```







































































































































































































































































































































































































































































































































































































