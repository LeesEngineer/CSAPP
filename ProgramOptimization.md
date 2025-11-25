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

<p><b>Removing Aliasing</b>: If I rewrite this code by introducing a local variable, it will gets a lot simpler.</p>

```
void sum_rows2(double *a, double *b, long n)
{
    long i, j;
    for(i = 0; i < n; i ++)
    {
        double val = 0;
        for(j = 0; j < n; j ++)
            val += a[i * n + j];
        b[i] = val;
    }
}
```

```
.L10:
    addsd    (%rdi), %xmm0
    addq     $8, %rdi
    cmpq     %rax, %rdi
    jne      .L10
```

<p>And again that's something that you as a programmer would hardly think is a big deal. But C compiler can't optimize for this in general. Because it can't determine in advance what possible aliasing there can be.</p>

<hr>

<p>So you need to get in the habbit of introducing local variables and using them, it's your way of telling compiler don't call the same function over and over again, don't read and write the same memory location over and over again just hold it in a temporary one. Then compiler will automatically allocate a register.</p>

</br>

# Exploiting Instruction-Level Parallelism

</br>

- Need general understanding of modern processor design

  - Hardware can execute multiple instructions in parallel
 
- Performance limited by data dependencies

- Simple transformations can yield dramatic performance improvement

  - Compilers often cannot make these transformations
 
  - Lack of associativity and distributivity in floating-point arithmetic

<p>I'm going to explain this by by a series of examples.</p>
 
</br>

## Data type for vectors

</br>

<p>Assume I have a data structure, that looks like the way pascal implements arrays.(I have nothing against pascal)</p>

<p>I used a data type called data_t. I can compile this code using different definitions of data_t to get int, long, float, double. We'll see how the performance characteristics shift with different data types.</p>

```
typedef struct
{
    size_t len;
    data_t *data;
} vec;

int get_vec_elemnet(vec *v, size_t idx, data_t *val)
{
    if(idx >= v -> len)
        return 0;
    *val = v -> data[idx];
    return 1;
}

void combine1(vec *v, data_t *dest)
{
    long int i;
    *dest = IDENT;
    for(i = 0; i < vec_kength(v); i ++)
    {
        data_t val;
        get_vec_element(v, i, &val);
        *dest = *dest OP val;
    }
}
```

<img width="2000" height="438" alt="QQ_1763885939717" src="https://github.com/user-attachments/assets/05722ba6-038f-49e0-a9dc-7fdf3b7f3806" />

<p><b>Basic Optimizations</b></p>

```
void combine4(vec *v, data_t *dest)
{
    long i = 0;
    long length = vec_length(v);
    data_t *d = get_vec_start(v);
    data_t t = IDENT;
    for(i = 0; i < length; i ++)
        t = t OP d[i];
    *dest = t;
}
```

- Move vec_length out of loop

- Avoid bounds check on each loop

- Accumulate in temporary

<img width="1460" height="390" alt="QQ_1763886364769" src="https://github.com/user-attachments/assets/958df8bd-8152-455b-addd-394380b26600" />

</br>

## Superscalar Processor

</br>

<p>Let's discuss how the numbers 1, 3, and 5 came about. You must have some understanding of the underlying hardware(ECE 741), These numbers indicate some fundamental limitation in your program.</p>

<p><b>Modern CPU Design(Simple version) : </b></p>

<img width="1844" height="1304" alt="QQ_1763887672039" src="https://github.com/user-attachments/assets/a2b6da25-f347-4292-90d6-793a296177a4" />

<p>The basic idea is you think a program as the computer just reads in an instruction and does whatever it says to do, reads in another instruction does what that says to do. CPU provide a massive hardware infrastructure to make program run faster than it was just doing one instruction at a time. It employs a technique that's called superscalar out of order execution. The idea is you think of your program as a linear sequence of instructions and CPU sucks in as many of instructions as it can. And CPU pulls instructions apart and realize that certain instructions don't really depend on each other, so I can start one even though it's later in the program than the one I'm working on right now.</p>

<p>As I mentioned as instruction level parallelism, what CPU tries to do is automatically detecting which instructions have no independencies, and then execute those instructions in parallel.</p>

<p>Your machine has resources to do multiple operations all at the same time. Those can only all get used by somehow structuring your program</p>

<p>It's called superscalar processor</p>

- Definition: A SP can issue and execute multiple instructions in one <b>cycle</b>. The instructions are retrieved from a sequential instruction stream and are usually scheduled dynamically

- Benefit: Without programming effort, SP can take advantage of the instruction level parallelism that most programs have

<p><b>Pipelined Functional Units: </b></p>

- Divid computation into stages

- Pass partial computations from stage to stage

- Stage i can start on new computation once values passed to i+1

<p>For example, multiple is more complex than you think, so you can break it into steps, and you have a separate dedicated hardware for each of those stages.</p>

```
long mult(long a, long b, long c)
{
    long t1 = a * b;
    long t2 = a * c;
    long t3 = t1 * t2;
    return t3;
}
```

<img width="1682" height="446" alt="QQ_1763895144144" src="https://github.com/user-attachments/assets/7e29a17a-6dbd-488f-849a-47094dba65d2" />

<p>'a*b' and 'a*c' don't depend on each other in any ways, so I can do them both but I don't have two hardware to do them simultaneously. Now p1*p2 obviously depends on previous operations.</p>

<p>Talk about Haswell CPU, it has 8 total functional units, combine these allows cpu to perform the most following operations at the same time (but not all, because there're some shared functional units) : </p>

- 2 load

- 2 store

- 4 integer operations

- 2 FP multiply

- 2 FP add

- 2 FP divide

<p>instructions has two characteristics. Latency is how long does a operation take. Because of pipelining the another called cycle represents the time interval between two micro-steps.(Some instructions take > 1 cycle, but can be pipelined.)</p>

<img width="1696" height="504" alt="QQ_1763968480707" src="https://github.com/user-attachments/assets/078cfe15-c23b-4d6f-ab59-c9f67976e854" />

<p>Notice that division is very slow, and it's not pipelined. Division is a very expensive operation</p>

<p>The two characteristics provide a limit on how fast our program can run.</p>

<hr>

<p>Integer Multiply: </p>

```
# Inner loop
.L519:
    imull    (%rax, %rdx, 4), %ecx
    addq     $1, %rdx
    cmpq     %rdx, %rbp
    jg       .L519
```

<p>I need the result of multiplication before I can begin the next, So there's a three clock cycle bound here.</p>

<img width="1458" height="434" alt="QQ_1763969532340" src="https://github.com/user-attachments/assets/7c0ff3b6-a828-457f-b859-d1727613e53f" />

<p>My measurements all correspond to what I'm calling the latency bound which is based on how much time it takes from a beginning of an operation to the end. </p>

<img width="1828" height="1206" alt="QQ_1763969826380" src="https://github.com/user-attachments/assets/a8def8cd-7a3f-457e-a328-74a469d2152a" />

<p>So that's why even though I have a pipelined multiplier, the program itself limits me to the sequential execution of all the multiplies</p>

<p>There's a fairly common technique called loop unrolling : Perform more useful work per iteration.</p>

```
void unroll_combine(vec *v, data_t *dest)
{
    long length = vec_length(v);
    long limit = length - 1;
    data_t *d = get_vec_start(v);
    data_t x = IDENT;
    long i;
    // Combine two elements at a time
    for(i = 0; i < limit; i += 2)
        x = (x OP d[i]) OP d[i + 1];
    for(; i < length; i ++)
        x = x OP d[i];
    *dest = x;
}
```

<p>When I run it, the integer addition got a little faster.</p>

<img width="1458" height="528" alt="QQ_1763971633350" src="https://github.com/user-attachments/assets/2ac5a205-e74a-4a35-aa2e-da2e903c1e0b" />

<p>But the other ones didn't improve at all, because they still have sequential dependency.</p>

<p>I could make a small change but change performance fairly dramatically.</p>

```
for(i = 0; i < limit; i += 2)
    x = x OP (d[i] OP d[i + 1]);
```

<img width="1460" height="772" alt="QQ_1763972171243" src="https://github.com/user-attachments/assets/081bd971-3344-47a6-a590-00c7afa98fcc" />

<p>'a' for associative transformation. Nearly 2x speedup for Int *, FP +, FP *. Because it breaks sequential dependency.</p>

<img width="754" height="824" alt="QQ_1763972382111" src="https://github.com/user-attachments/assets/07adaf25-5abb-44a2-8cdc-3f8fc890e806" />

<p>What changed: Ops in the next iteration can be started early(No dependency)</p>

<p>Overall performance: </p>

- N elements, D cycles latency / op

- (N / 2 + 1) * D cycles: CPE = D / 2

<p>The good news is you know that two's complement arithmetic is associative and commutative, so it doesn't matter for both multiplication and addition, you'll get the exact same answer no matter what order you combined these integers. But floating-point isn't associative, If you shift these parentheses, because of rounding possibilities and even potential overflow, you might get different values.</p>

<p>Even though rounding is infrequent, it's enough that the compiler won't change the associativity of floating-number. They are very conservative when it comes to floating-point.</p>

<p>There's a more fundamental bound called throughput bound represents how many "same type of operations" can the CPU complete per cycle which is based on the number of functional units. </p>

<hr>

<p><b>Loop Unrolling with Separate Accumulators</b></p>

```
void unroll2_combine(vec *v, data_t *dest)
{
    long length = vec_length(v);
    long limit = length - 1;
    data_t *d = get_vec_start(v);
    data_t x0 = IDENT;
    data_t x1 = IDENt;
    long i;
    for(i = 0; i < limit; i += 2)
    {
        x0 = x0 OP d[i];
        x1 = x1 OP d[i + 1];
    }
    for(; i < length; i ++)
    {
        x0 = x0 OP d[i];
    }
    *dest = x0 OP x1;
}
```

<p>This transformation I called multiple accumulators breaks out the latency limitation and get close to throughput. It's a different form of reassociation to get more parallelism</p>

<img width="1892" height="764" alt="QQ_1763978481647" src="https://github.com/user-attachments/assets/596cdfde-fe80-48da-b6a3-408e31fcaa0a" />

<img width="1044" height="786" alt="QQ_1763979317184" src="https://github.com/user-attachments/assets/e44ff22e-5ad0-4e21-a346-1da51ebc4fbf" />

<p>We can generalize this, we can unroll by a factor of K to any degree L, L must be multiple of K. </p>

<img width="1622" height="954" alt="QQ_1763979659815" src="https://github.com/user-attachments/assets/c16877ea-659e-47ba-8c18-c88ffe17f4e4" />

<p>In general, by picking the best parameter, I can get very close to the throughput bound of the processor. The CPE of the program originally was 20 clock cycles, now I'm getting it down to one.</p>

<hr>

<p>We can reduce CPE even further.</p>

<p>When I talk about floating-point I mentioned that there's a special set of registers called %xmm. Now the new generation of CPU have something called %ymm, 16 total, each 32 bytes.</p>

<p>you can think of these as a way of operating on 32 individual characters.(Data-level parallelism)</p>

<img width="1780" height="1210" alt="QQ_1763980427042" src="https://github.com/user-attachments/assets/d975ea5d-1327-4ebe-8d33-864ff6f68420" />

<p>There're also instructions called vector operations. You can do eight or four floating-point multiplications in parallel and pipelined in three clock cycles.</p>

<p>If I use what I called vector code, you can see that CPE has decreased by four times(For add).</p>

<img width="1896" height="722" alt="QQ_1763981839984" src="https://github.com/user-attachments/assets/33454d75-8d57-43f9-a913-6cc9cb574ae8" />

<p>So make use of AVX instructions(Parallel operations on multiple data elements)</p>

<p>Intel compiler will automatically do some of this for you. GCC attempted to implement it but it didn't work well, so I think they discontinued that.</p>

</br>

# Dealing With Conditionals

</br>

<p>Challenge:</p>

- Instruction control Unit must work well ahead of Execution Unit to generate enough operations to keep EU busy

- When encounters conditional branch, cannot reliably determine where to continue fetching

<p>To simplify, you can think of your program as long linear sequence of instructions. CPU trying to grab as many of instructions as they can, and pull them apart.</p>

<p>Some program is a loop, and there're not many instructions in the loop. So how to turn it into a linear sequence. That relies on how do you handle branches.</p>
![Photo on 2025-11-24 at 22 32 #2](https://github.com/user-attachments/assets/58ba7478-bdaa-4026-8b51-27d66b657504)





























































































































































































































































































































































































































































































