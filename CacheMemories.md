</br>

# Cache Memories

</br>

<p>Cache memories are fast SRAM-based memories managed automatically in hardware, and hold frequently accessed blocks of main memory.</p>

<img width="1556" height="590" alt="QQ_1766047141461" src="https://github.com/user-attachments/assets/6e402a59-7623-406f-bc27-9d5cb9ac35fe" />

<p>Most of our requests for data will actually be served out of this cache memory and cost a few cycles rather than the slow main memory.</p>

<p>Hardware logic has to know how to look for blocks in cache, and determine whether or not a perticular block is contained here. So cache memory has to be organized in a strict way.</p>

<p>The whole purpose of blocks is to exploit spatial locality.</p>

<p>All cache memories are organized in the following way.</p>

<img width="1404" height="1038" alt="QQ_1766066931479" src="https://github.com/user-attachments/assets/dbe6f282-1903-4622-a21b-bf5c25a4c34e" />

<p>Each cache has S sets. There're E blocks per set and there're B bytes per block.</p>

<p>A valid bit indicates whether the data bits and data blocks are actually meaningful. It's possible that they could just be random bits when you firstly turn on the machine.</p>

<p>There's some additional bits called tag bits which help us search for blocks.</p>

</br>

# Cache Read

</br>

<p></p>

<p>In the x86-64 architecture, <b>the CPU assigns a 64-bit to each byte in memory</b>. When program executes an instruction that references some word in memory. CPU sends that address to the cache and asks the cache to return the word at that address. But cache divides the address into a number of regions instead of using them directly</p>

<img width="516" height="330" alt="QQ_1766073769056" src="https://github.com/user-attachments/assets/674c2eab-c8e7-4383-9b17-7d503d99cb73" />

<p>There are b low order bits which determine the offset in the block that word starts at. For example, if a cache block is 64 bytes, then log2(64) = 6 bits are needed.<b>Note that the offset doesn't participate in cache lookup, it only determines where in the block you retrieve data.</b></p>
  
<p>The next s bits are treated as an unsigned integer which serves as an index into the array of sets. And then all of the remaining t bits constitude what we call tag which determines whether this block is what we need</p>

<p>The cache logic takes this address, and it first extracts the set index to identify the set. If the block that contains the data word at this address is in the cache. It's going to be in the array donoted by the set index.</p>

<p>And then it checks the tag. it checks all of the lines in that set to see if there's any of those lines have a matching tag.</p>

- Locate set

- Check if any line in set has matching tag (CPU performs parallel comparisions)

- <b>Yes + line valid: hit</b>

<p>If the tag doesn't match the old-line, then there's a miss. And in that case, the cache has to fetch the corresponding block from memory and then overwrite this block in the line. Then it can get the word out of the block and send it back to the processor</p>

</br>

## Direct-Mapped Cache Simulation

</br>

<img width="1914" height="244" alt="39ea7150029b8eaa1cbe4be3294ea03e" src="https://github.com/user-attachments/assets/3fe7c374-16ec-49dc-8583-491c27ac5851" />

<p>This is a very simple system</p>

```
Address trace (reads, one byte per read)
0        [0000]        miss
1        [0001]        hit
7        [0111]        miss
8        [1000]        miss
0        [0000]        miss
```

<p>First, it extracts the set index bits which in this case are 00. Since valid bit is 0 it's just a miss. And set the tag bits to 0. In case of address 1, this is a hit, because the block that contains the byte at address 1 is already in the cache, and tag matches good.</p>

<p>Now we get address 7, cache extracts the set index bits, Looks in set 3, this is miss. <b>It loads the data from memory</b></p>

<p>The next reference that comes by is 8 which has set index of 00, but that's currently occupied by the block M[0-1]</p>

<img width="828" height="432" alt="QQ_1766128717955" src="https://github.com/user-attachments/assets/bf1a162d-98c7-49dc-be19-6d7808743a00" />

<p>Note that M[6-7] is used here since blocks are two bytes and they'll start on an even multiple</p>

<p>Address 8 has a tag of 1, so this is a miss. So we have to fetch the block containing byte number 8 into cache</p>

<img width="934" height="442" alt="QQ_1766129201446" src="https://github.com/user-attachments/assets/3dc49945-437d-4202-8a4b-8c0eeec3a1f7" />

<p>For address 0, it's another miss so that's unfortunate.</p>

<p>The only reason we missed it is because we just have one line per set, so we were forced to overwrite.</p>

<p>There's plenty of room in cache. Just because of the low associativity of cache and the access pattern, we've got a miss that really was kind of unnecessary.</p>

<p>This is the reason why caches have higher associativity, higher values of E. For values of E greater than 1, we refer to them as E way associative caches.</p>

<img width="1386" height="650" alt="QQ_1766156506012" src="https://github.com/user-attachments/assets/fe38f0fc-9b38-45e6-8a06-187ec3d99227" />

<p>Here is a 2-way associative cache. We have address. The cache extracts that set index and throws away all the other sets.</p>

<p>Now cache is searching for a matching tag in both of these lines <b>in parallel</b>. Okey Once we've identified a match tag, then we use set offset bits. We just assume that the cache knows the what size to return.</p>

<hr>

<p>As the associativity goes up, that logic gets more and more expensive. If you take this to the limit there's just one set which is called <b>fully associative cache</b>, and now a block can go anywhere.(There's no constraints on where you place a block) But because of the complexity of the fully associative search, those are very rare. </p>

<p>In a virtual memory system, the DRAM serves as a cache for data stored on the disk. If you hava a cache on DRAM and you got a miss, then you have to go to disk. The penalty is huge for that. And because of that, it's worth while having very complex search algorithms. In particular in a virtual memory system, DRAM implements a fully associative cache where blocks from disk can goto anywhere.</p>

<p>The largest associativity that I know is Intel systems which is 16-way associative L3 caches</p>

</br>

# Cache Write

</br>

<p>We're creating subsets of the data in caches as we move up in the hierarchy. Multiple copies of data exists in L1, L2, L3, main memory and disk.</p>

<p>What to do on a write-hit. We have to options: </p>

- Write-through (write immediately to memory)

- Write-back (defer write to memory until replacement of line)

  - Need a dirty bit (line different from memory or not)

<p>In the first option, memory always mirrors the contents of cache. But that's expensive. I mean you know memory accesses are expensive. In the case of the other option, we write to a block in cache and <b>don't flush it to memory until we elect that particular line as a victim</b>. The algorithm is when the cache identifies a particular line is going to be overwritten, it checks the dirty bit on that line. If it's set then cache writes that data back to disk.</p>

<p>What to do on a write-miss: </p>

- Write-allocate: load into cache, updata line in cache

  - Good if more writes to location follow
     
- No-write-allocate: writes straight to memory, does not load into cache

<p>Different caches use different policies. Typically we use write-through + no-write-allocate or write-back + write-allocate</p>

</br>

# Cache Hierarchy

</br>

<p>So far we've only assumed that there's a single cache. But in a real system, there're multiple caches.</p>

<p>The following is Intel core i7 cache hierarchy which contains multiple processor cores. These processor cores can each execute their own independent instruction stream in parallel. And each processor core can contain general-purpose registers and two different kinds of L1-cache(d-cache is data cache, i-cache is instruction cache).</p>

<img width="1210" height="1028" alt="QQ_1766224495764" src="https://github.com/user-attachments/assets/7dab7918-2bff-46ed-be58-f5e03c7f85d5" />

<img width="488" height="604" alt="QQ_1766225146808" src="https://github.com/user-attachments/assets/121c49f1-a47e-4b9c-a268-827863e3ef4f" />

</br>

# Writing Cache Friendly Code

</br>

<p>Caches are automatic. And they're all built in hardware, so there's no the sort of visible instruction set that lets you manipulate caches and your assembly machine code programs. It all happens behind the scenes a utomatically in hardware. If you have a general idea of how it works, then you can write code that's cache friendly</p>

<p>Minimize the misses in the inner loop: </p>

- Repeated references to variables are good (Temporal locality)

<p>Especially if those are local variables. Remember if you declare a local variable, the compiler can put that in a register. If you're referencing global variables, the compiler doesn't know what's going on. <b>Because the complier cannot be certain whether this global variable will be modified elsewhere, </b> it can't put the reference to that variable in a register. So repeated references to local variables stored on the stack are good. Those will get turned into register accesses, you'll never go to memory.</p>

- Stride-1 reference patterns are good (Spatial locality)

</br>

# Performance impact of caches

</br>

## The Memory Mountain

</br>

- Read throughput (Read bandwidth)

  - Number of bytes read from memory per second
 
- Memory mountain: Measured read throughput as a function of spatial and temporal local.

  - Compact way to characterize memory system performance

<p><b>Memory Mountain Test Function: </b></p>

```
long data[MAXELEMS];

int test(int elems, int stride)
{
    long i, sx2 = stride * 2, sx3 = stride * 3, sx4 = stride * 4;
    long acc0 = 0, acc1 = 0, acc2 = 0, acc3 = 0;
    long length = elems, limit = length - sx4;

    for(i = 0; i < limit; i += sx4)
    {
        acc0 = acc0 + data[i];
        acc1 = acc1 + data[i + stride];
        acc2 = acc2 + data[i + sx2];
        acc3 = acc3 + data[i + sx3];
    }

    for(; i < length; i ++)
        acc0 = acc0 + data[i];

    return ((acc0 + acc1) + (acc2 + acc3));
}
```

<p>If you write in the lollowing way. The result of each operation depends on the previous one, and pipline will be blocked by these dependencies. CPU can't execute them in parallel. So I'm actually doing sort of four scans in parallel.</p>

```
acc += data[i];
acc += data[i+stride];
acc += data[i+2*stride];
acc += data[i+3*stride];
```

<p>Call test() with many combinations of elems and stride. For each elems and stride: </p>

- <b>Call test() once to warm up the caches</b>

- Call test() again and measure the read throughput (MB / s)

<img width="1322" height="988" alt="QQ_1766404631503" src="https://github.com/user-attachments/assets/ce129dcf-aa41-4cdf-9e95-4d20eb6c2e88" />

<p>A beautiful picture. As we increase stride we're decreasing the spatial locality. As we increase the size we're decreasing the impact of temporal locality because as we increase the size there's fewer and fewer caches that can hold data.</p>

</br>

## Rearranging loops to improve spatial locality

</br>

<p>One way to improve the spatial locality is to rearrange loops, and I'll use matrix multiplication as an example.</p>

```
/* ijk */
for(i = 0; i < n; i ++)
    for(j = 0; j < n; j ++)
    {
        sum = 0.0;    // Variable sum held in register
        for(k = 0; k < n; k ++)
            sum += a[i][k] * b[k][j];
        c[i][j] = sum;
    }
```

<p>We can permute these loops. Five other possibilities are feasible. We can analyze these different permytations and predict which one will have the best performance.</p>

<p>Assume: </p>

- Block size = 32B (big enough for four double)

- Matrix dimension (N) is very large

- Cache is not even big enough to hold multiple rows

<p>The Analysis method is to <b>look at access pattern of inner loop.</b> We always focus on inner loop.</p>

<p>For ijk:</p>

<img width="644" height="622" alt="QQ_1766461301181" src="https://github.com/user-attachments/assets/836a3087-0ba0-4f26-87b3-873bc91f8c08" />

<p>Assuming that we can hold four integer elements in one block, row wise access which has a good spatial locality will miss one every four accesses. Because the access pattern for b is column wise, every reference to b will miss.</p>

<img width="808" height="224" alt="QQ_1766462514588" src="https://github.com/user-attachments/assets/33885136-3492-4a55-93a2-ded4939d10f6" />

<p>The average number of misses per loop iteration is 1.25.</p>

```
/* kij */
for(k = 0; k < n; k ++)
    for(i = 0; i < n; i ++)
    {
        r = a[i][k];    // Variable sum held in register
        for(j = 0; j < n; j ++)
            c[i][j] += r * b[k][j];
    }
```

<img width="716" height="494" alt="QQ_1766462197829" src="https://github.com/user-attachments/assets/93979f40-31a3-4246-859b-487878d9af59" />

<img width="844" height="252" alt="QQ_1766462580633" src="https://github.com/user-attachments/assets/8a105ea9-d28b-4625-aea9-69423b5ada16" />

<p>The average number of misses per loop iteration is 0.5.</p>

```
/* jki */
for(j = 0; j < n; j ++)
    for(k = 0; k < n; k ++)
    {
        r = b[k][j];    // Variable sum held in register
        for(i = 0; i < n; i ++)
            c[i][j] += a[i][k] * r;
    }
```

<img width="664" height="590" alt="QQ_1766462725609" src="https://github.com/user-attachments/assets/a09f70bf-28e3-43c7-beb7-853130920be8" />

<p>The average number of misses per loop iteration is 2.</p>

<p>Summary: </p>

- ijk (& jik)

  - 2 loads, 0 stores
 
  - misses / iter = 1.25
 
- kij (& ikj)

  - 2 loads, 1 store
 
  - misses / iter = 0.5
 
- jki (& kji)

  - 2 loads, 1 store
 
  - misses / iter = 2.0
 
<p>It' looks like kij and its brother are the best. but kij has this additional store. Writes have a lot more flexibility than reads. You can defer writing until the value that you're written is actually used. But when you read an item you're stuck, you can't do anything until you get that data. So this additional store doesn't really hurt us.</p>

<img width="1646" height="982" alt="QQ_1766470457021" src="https://github.com/user-attachments/assets/71bf19ec-c40f-444f-baf9-7aad033789f9" />

</br>

## Using blocking to improve temporal locality

</br>

<p>Blocking is a very general technique</p>

```
c = (double *) calloc(sizeof(double), n * n)

void mmm(double *a, double *b, double *c, int n)
{
    int i, j, k;
    for(i = 0; i < n; i ++)
        for(j = 0; j < n; j ++)
            for(k = 0; k < n; k ++)
                c[i*n + j] += a[i*n + k] + b[k*n + j];
}
```

<img width="900" height="258" alt="QQ_1766471840625" src="https://github.com/user-attachments/assets/e4abfe2e-5121-488b-8ec2-a92905b33bf4" />

<p>Assuming that cache block can hold 8 doubles. Total misses = (n/8 + n) * n^2 = (9/8) * n^3.</p>

<p>Let's write the code to use blocking: What we're doing instead of updating one element at a time, we're updating a B by B sub block. This B by B sub block in c is computed by taking the inner product of sub blocks. For each one we're doing a little mini matrix multiplication.</p>

```
c = (double *) calloc(sizeof(double), n*n);

void mmm(double *a, double *b, double *c, int n)
{
    int i, j, k;
    for(i = 0; i < n; i += B)
        for(j = 0; j < n; j += B)
            for(k = 0; k < n; k += B)
                /* B * B mini matrix multiplication */
                for(i1 = i; i1 < i + B; i1 ++)
                    for(j1 = j; j1 < j + B; j1 ++)
                        for(k1 = k; k1 < k + B; k1 ++)
                            c[i1 * n + j1] += a[i1 * n + k1] * b[k1 * n + j1];
}
```

<img width="1206" height="412" alt="e2007a7950d095643f2095b6e16575ca" src="https://github.com/user-attachments/assets/20f90f8c-1e78-45f8-943a-d09f7a772e90" />

<p>There're B^2/8 misses for each block. And 2n/8 * B^2/8 = nB/4. So total misses = nB/4 * (n/B)^2 = 1/(4B) * n^3</p>

<p>Suggest largest possible block size B, but limit 3B^2 < 2.</p>

<p>Reason for dramatic difference: </p>

- Matrix multiplication has inherent temporal locality

  - Input data: 3n^2, computation 2n^3
 
  - Every array elements used O(n) times
 
- But program has to be written properly

<hr>

<p>Cache memories are built-in automatic hardware storage devices. you can't control them, but you can exploit them and make your code run fast.</p>

- Try to maximize spatial locality by reading data objects with sequentially with stride 1

- Try to maximize temporal locality by using a data object as often as possible once it's read from memory







































































































































































































































































































































































































































































































































































































































































































































































