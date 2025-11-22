</br>

# Overview

</br>

<p>The main idea is how can I make programs run fast.</p>

<p>Optimizing Compilers</p>

- Provide efficient mapping of program to machine

  1. register allocation
 
  2. code selection and ordering (scheduling)
 
  3. dead code elimination
 
  4. eliminating minor inefficiencies

- Don't (usually) improve asymptotic effeciency

  1. up to programmer to select best overall algorithm
 
  2. big-O savings are (often) more important than constant factors

- Have difficulty overcoming "optimization blocks"

  1. potential memory "aliasing"
 
  2. potential procedure side-effect

<p>Limitations of optimizing compiler : Compiler has a whole cookbook of optimization strategies. But in general if compiler feels like the code is something that it doesn't feel confident about being able to make certain transformations, it just won't and it will keep things a more direct implementation of exactly what you described.</p>

- Operate under fundamental constraint

  1. Must not cause any change in program behavior. Except, possibly when program making use of nonstandard language features

  2. Often prevents it from making optimizations that would only affect behavior under pathological conditions.

- Behavior that may be obvious to the programmer can be obfuscated by languages and coding styles

  1. e.g., Data ranges may be more limited than variable types suggested
 
- Most analysis is performed only within procedures

  1. Whole-program analysis is too expensive in most cases
 
  2. Newer versions of GCC do interprocedural analysis within individual files, but not between code in different files
 
- Most analysis is based only on static information

  1. Compiler has difficulty anticipating run-time inputs
 
- When in doubt, the compiler must be conservative

</br>

# Generally Useful Optimizations

</br>

<p>Optimizations that you or the compiler should do regardless of processor / compiler</p>

- Code Motion

  - Reduce frequency with which computation performed
 
    1. If it will always produce same result
   
    2. Especially moving code out of loop
   
<img width="1730" height="412" alt="QQ_1763789711384" src="https://github.com/user-attachments/assets/77743e38-20d8-4908-9593-e103e90b094d" />

- Reduction in Strength

  - Replace costly operation with simpler one
 
  - Shift, add instead of multiply or divid : On Intel Nehalem, integer multiply requires 3 CPU cycles
 
  - Recognize sequence of products
 
<img width="1682" height="342" alt="QQ_1763790179880" src="https://github.com/user-attachments/assets/e617da77-698f-4225-9b64-0143913528a7" />

- Share Common Subexpressions

  - Reuse portions of expressions
 
  - GCC will do this with -O1

<img width="1776" height="990" alt="QQ_1763790349368" src="https://github.com/user-attachments/assets/acad6865-486e-4060-8d31-6b406ff87d5d" />
 
<p></p>

</br>

# Optimization Blockers

</br>

</br>

## #1 Procedure Calls

</br>

<p>Procedure to convert string to lower Case : </p>

```
void lower(char *a)
{
    size_t i;
    for(i = 0; i < strlen(s); i ++)
        if(s[i] >= 'A' && s[i] <= 'Z')
            s[i] -= ('A' - 'a');
}
```

<p>I was horrified about this code.</p>

- Time quadruples when double string length

- Quadratic performance

<img width="1970" height="842" alt="QQ_1763791428965" src="https://github.com/user-attachments/assets/62fdfa87-1edc-498e-a3e3-db862276b715" />

<p>This is a kind of program that easily have some hidden performance bugs, that makes them run quadratic.</p>

<p>The key is when in a test like the calling of strlen, it's determining whether it reaches the end of the string to figure out how long the string is.</p>

<p>If we do the conversion of a loop into a goto form. Strlen executed every iteration.</p>

```
void lower(char *a)
{
    size_t i = 0;
    if(i >= strlen(s))
        goto done;
loop:
    if(s[i] >= 'A' && s[i] <= 'Z')
        s[i] -= ('A' - 'a');
    i ++;
    if(i < strlen(s))
        goto loop;
done:
}
```

<p>You should introduce a local variable to precompute the length. </p>

<p>So why couldn't a compiler figure this out.</p>

- Why couldn't compiler move strlen out of inner loop

  - Procedure may have side effects : Alters global state each time called
 
  - Function may not return same value for given arguments
 
- Warning

  - Compiler treats procedure call as a block box
 
  - Weak optimizations near them
 
- Remedies

  - Use of inline functions : GCC does this wih -O1, within single file
 
  - Do your own code motion
 
</br>

## Memory Matters

</br>

```
void sum_rowsl(double *a, double *b, long n)
{
    long i, j;
    for(i = 0; i < n; i ++)
    {
        b[i] = 0;
        for(j = 0; j < n; j ++)
            b[i] += a[i * n + j];
    }
}
```

```
# inner loop
.L4:
    movsd    (%rsi, %rax, 8), %xmm0
    addsd    (%rdi), %xmm0
    movsd    %xmm0, (%rsi, %rax, 8)
    addq     $8, %rdi
    cmpq     %rcx, %rdi
    jne      .L4
```

<p>Code updates b[i] on every iteration.</p>

<p>The main thing is it's reading from memory, and it's adding something to it then it's writing back to memory. That memory location corresponds to b[i].</p>

<p>Why couldn't compiler optimize this away? Why does it have to keep jumping back and forth between memory and registers over and over again.</p>

<hr>

<p>The reason is in C you can't be sure whether there's what known as aliasing. That's referred to as aliasing when separate parts of program are referring to the same location in memory</p>

<p>Imagine aliasing happened when 'b' can corresponds to a row of array 'a'. It demonstrates that what will happen is as b gets updated, it's changing a and it's changing what's being read during summation.</p>

<p>So when compiler's given code like this, it has to assume that these two memory locations might overlap each other. So that's why it's carefully reading it out and writing it back.</p>

<hr>

<p>If I </p>

</br>

# Exploiting Instruction-Level Parallelism

</br>

<p></p>

</br>

# Dealing With Conditionals

</br>

<p></p>





































































































































































































































































































































































































































































































