</br>

# Exceptions and Processes

</br>

## Exceptional Control Flow

</br>

<p>Processors only do one thing: From startup to shutdown, a CPU simply reads and excutes a sequence of instructions. This sequence is the CPU's control flow.</p>

<p>Up to now, there're two mechanisms for changing control flow: Jumps and branches, call and return. <b>They are both reacting to changes in program state (check the control condition codes)</b></p>

<p>But it's insufficient for a useful real system, it's difficult to react to changes in system state: </p>

- Data arrives from a disk or a network adapter

- Instruction divides by zero

- User hits Ctrl-C at the keyboard

- System timer expires

<p>All of these represent some changes in the system state. So system need some mechanisms for "execptional control flow". ECF exists at all levels of a computer system: </p>

- Low level mechanisms

  - Exceptions
 
    - Change in control flow in reponse to a system event (i.e., change in system state)
   
    - Implemented using combination of hardware and OS software
   
- Higher level mechanisms

  - Process context switch
 
    - Implemented by OS software and hardware timer
   
  - Signals
  
    - Implemented by OS software
   
  - Nonlocal jumps: setjmp() and longjmp()
 
    - Implemented by C runtime library

<p>Nonlocal jumps allow you to break the normal call and return pattern.</p>

</br>

## Exceptions

</br>

<p>An exception is a transfer of control to the OS kernel in reponse to some event (i.e., change in processor state)</p>

- <b>Kernel is the memory-resident part of the OS</b>

- Examples of events: Divide by 0, arithmetic overflow, page fault, I/O request completes, typing Ctrl-C

<img width="1252" height="534" alt="QQ_1767251211892" src="https://github.com/user-attachments/assets/a1e46a06-757f-47ef-96d1-c0eebb212806" />

<p>Exceptions are implemented by hardware and software. The transfer of control you know the change in the program counter or %rip is donw by the hardware. <b>And the code that executes as a result of that exception is set up and determined by the kernal.</b></p>

<p><b>Every type of event has a unique exception number which serves as an index into a jump table called an exception table</b>. When event k happens, the hardware uses k as an index into the table, and gets the address of the exception handler for this exception. Handler k is called each time exception k occurs.</p>

<img width="826" height="764" alt="QQ_1767251777163" src="https://github.com/user-attachments/assets/b90f531a-acd8-4cd7-8774-ab555836bfa8" />

<hr>

<p>There're different kinds of exceptions we distinguish them as asynchronous and synchronous.</p>

<p>Asynchronous exceptions happen as a result of changes in state that are occurred outside of the processor. This exception was not caused by "the instruction currently being executed".: </p>

- Caused by events external to the processor

- Indicated by setting the processor's interrupt pin

- Handler returns to "next" instruction

<p>For an interrupt, there's like a little pause while the interrupt handler runs and then your program just continues to run. It's done behind the scenes and doesn't affect your execution of the program.</p>

<p>Example: </p>

- Timer interrupt

  - Every few ms, an external timer chip triggers an interrupt
 
  - Used by the kernel to take back control from user programs
 
- I/O interrupt from external device

  - Hitting Ctrl-C at the keyboard
 
  - Arrival of a packet from a network
 
  - Arrival of data from a disk

<p><b>All systems have a built-in timer.</b> When the timer goes off it sets the interrupt pin high. We need this to allow the kernel to get control of the system again. And then kernel can dicide what to do, <b>maybe schedule a new process or let the current process run</b>. Otherwise a user program could just run forever in an infinite loop, and there's no way for the system to get control.</p>

<hr>

<p>The synchronous exception is caused by event that occur as a result of executing an instruction. There're three classes: </p>

- Traps

  - it's an intentional exception. it's an exception where "the program actively requests the CPU to enter the kernel".
 
  - Examples: System calls, breakpoint traps, special instructions
 
  - Returns control to "next" instruction
 
- Faults

  - Unintentional but possibly recoverable
 
  - Examples: page faults (recoverable), protection faults (uncoverable), floating point exceptions
 
  - Either re-executes faulting ("current") instruction or aborts
 
- Aborts

  - Unintentional and unrecoverable
 
  - Examples: illegal instruction, parity error, machine check
 
  - Aborts current program

<p>The most common form of a trap is a system call. The kernel provides all kinds of services to a program. But your program can't access directly, it can't call functions in the kernal and can't access data directly in the kernal. Because <b>this part of memory is unavailable to user.</b> So kernel provides an interface that allows program to make requests to effectively call functions in the kernel. It looks like a function call but it's really transferring control into the kernal.</p>

</br>

## Exception in Linux/x86-64 System

</br>

<p>Each x86-64 system call has a unique ID number assigned by Linux.</p>

<img width="1246" height="704" alt="QQ_1767262685486" src="https://github.com/user-attachments/assets/e1ae3829-bdb8-4d40-a64a-ecd7010af40b" />

<P>There's an instruction called syscall which actually performs the system call. You don't strike the syscall instruction directly in your program. Linux wraps those in system level functions. It's interesting in C how it works. Suppose you want to open a file: </P>

- User calls open(filename, options)

- Calls __open function, which invokes system call instruction syscall

```
000000000000e5d70 <__open>:
...
e5d79:  b8 02 00 00 00        mov    $0x2,%eax      # open is syscall #2
e5d7e:  0f 05                 syscall               # Return value in %rax
e5d80:  48 3d 01 f0 ff ff     cmp    $0xfffffffffffff001,%rax
...
e5dfa:  c3                    retq
```

- %rax contains syscall number

- Other arguments in %rdi, %rsi, %rdx, ...

- Return value in %rax

- If %rax is negative then it means some error occurred

<p>You can see the code is checking the return value</p>

<img width="854" height="436" alt="39223fbebc867049582bc5065b7429fd" src="https://github.com/user-attachments/assets/a5511d46-2b23-4d67-9133-1e8ce05e4250" />

<hr>

<p>Lat'e look at an example of a fault. It's a page fault.</p>

```
int a[1000];
main()
{
    a[500] = 13;
}
```

- User writes to valid memory location

- That portion (page) of user's memory is currently on disk

`80483b7:        c7 05 10 9d 04 08 0d    movl    $0xd, 0x8049d10`

<P>For this movl instruction, because the memory at this address isn't available, it will trigger a page fault and transfer the control into the page fault handler in the kernel which copies that page from disk to memory. <b>And when it returns it will re-execute this movl.</b> Now It's available.</P>

<img width="990" height="432" alt="QQ_1767334596977" src="https://github.com/user-attachments/assets/fbf87f78-f115-4dde-9484-d33b9149400d" />

<hr>

<p>Another fault is an invalid memory reference</p>

```
int a[1000];
main()
{
    a[5000] =13;
}
```

`80483b7:        c7 05 60 e3 04 08 0d    movl    $0xd, 0x804e360`

<p>It's an invalid reference. In this case <b>it looks like a page fault, but the kernel detects that it's an invalid address</b> there isn't anything that can be loaded from disk. <b>So it sends a signal to the process</b> and then never returns.</p>

<img width="1212" height="382" alt="QQ_1767335200315" src="https://github.com/user-attachments/assets/65e1a373-c128-4c63-a072-8606afc7114b" />

- Sends <b>SIGSEGV</b> signal to user process

- User process exits with segmentation fault

</br>

## Processes

</br>

<p>We've seen exceptions or low-level transfers of control. The higher level is another form of exception control flow called process.</p>

<p>Process is an instance of a running program. Process provides each program with two key abstraction: </p>

- Logical control flow

  - Each program seems to have exclusive use of CPU
 
  - Provided by kernel mechansim called context switch
 
- Private address space

  - Each program seems to have exclusive use of main memory
 
  - Provided by kernel mechanism called virtual memory

<p>It looks like you have unique exclusive use of the processor and registers, and each running program has its own code, data, heap, stack.</p>

<img width="1008" height="586" alt="QQ_1767337351038" src="https://github.com/user-attachments/assets/6b36e3ee-1dd5-4d4b-84d2-87f7565cb873" />

<p>Computer runs many processes simultaneously.(Even on a system with a single core) </p>









</br>

## Process Control

</br>

<p></p>


































































































































































































































































































































































































































































































































































































