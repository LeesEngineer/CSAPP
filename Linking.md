</br>

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

## Object Files (Modules)

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

### Executable and Linkable Format (ELF)

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

## Linker Symbols

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

<p>But if we declare these weak symbols with different types, we will get into trouble. <b>Writes to x in p2 might overwrite y!</b> </p>















</br> 

# Library interpositioning

</br>

<p>This technique allows you to use linking to actually intercept on function calls in libraries like the standard C library. It's all enabled by linking.</p>



<p></p>




















































































































































































































































































































































































































































































































































































































