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

<p>Computer runs many processes simultaneously.(Even on a system with a single core). Process executions are interleaved.</p>

<hr>

<p>Suppose program is running on a single core system. At some point an exception occurs either because of a timer interrupt or a fault of some kind or a trap, <b>the operating system will gets control.</b> It will dicide which process to run.</p>

<img width="1060" height="774" alt="QQ_1767337974273" src="https://github.com/user-attachments/assets/ceee4acb-6dde-44c7-a742-7ed888facef9" />

- Save current registers in memory

- Schedule next process for execution

- Load saved registers and switch address space(context)

<p>But now operating system will schedule processes on those multiple cores. If there's not enough cores then it will do the context switch.</p>

</br>

### Concurrent Processes

</br>

<p>Each process is a logical control flow. Two processes run concurrently if their flows overlap in time, otherwise they are sequential. Control flows for concurrent processes are disjoint, but we can think of concurrent processes as running parallel.</p>

<p>Processes are managed by a shared chunk of memory-resident OS code called the kernel. <b>The kernel is not a separate process, but rather runs as part of some existing processes.</b> It's just code that's in the upper portion of the address space. <b>Kernel gets executed as a result of an exception.</b></p>

<p>Control flow passes from one process to another via a context switch.</p>

<img width="1220" height="612" alt="QQ_1767341354025" src="https://github.com/user-attachments/assets/d7874710-c82c-4de6-8639-70fa7cb6c3f2" />

<p>You have this process A that's runs, and then an exception occurs which transfers control to the kernel. Kernel invokes its scheduler which decides whether to let A continue to run or to do a context switch and run a new process.</p>

</br>

## Process Control

</br>

<p>Linux provides a number of functions that you can call to manipulate processes that we refer to as process control.</p>

<p>On error, Linux system-level functions typically return -1 and set global variable srrno to indicate cause.</p>

<p><b>Hard and fast rule: </b>You must check the return status of every system-level function. Only exception is the handful of functions that return void.</p>

```
if((pid = fork()) < 0)
{
    fprintf(stderr, "fork error: %d\n", strerror(errno));
    exit(0);
}
```

<p>You can also make a error-handling wrappers</p>

<hr>

<p>Obtain PID: </p>

- pid_t getpid(void)

  - Returns PID of current process
 
- pid_t getppid(void)

  - Returns PID of parent process

</br>

### Creating and Terminating Processes

</br>

<p>From a programmer's perspective, we can think of a process as being in one of three states</p>

- Running: Process is either executing, or waiting to be executed and will eventually be scheduled by the kernel

- Stopped: Process execution is suspended and will not be scheduled until further notice

- Terminated: Process is stopped permanently

<p>Process can be terminated by three reasons: </p>

- Receiving a signal whose default action is to terminate

- Returning from the main routine

- Calling the exit function

<p>`exit` is called once but never returns. `void exit(iont status)` terminates with an exit status. . Another way to explicitly set the exit status is to return an integer value from the main routine.</p>

<p>Convention: normal return status is 0, nonzero on error.</p>

<hr>

<p>Parent process creates a new running process by calling fork. `int fork(void)`</p>

- Returns 0 to the child process, child's PID to parent process

- Child is almost identical to parent

  - Child gets an identical <b>(but separate)</b> copy of the parent's virtual address space
 
  - Child gets identical copies of the parent's open file descriptors
 
  - Child has a different PID than the parent
 
<p>`fork` is called once but returns twice. Your code looks like this: </p>

```
pid_t pid = fork();
```

<p>This line of code was executed only once. But the program was "copied". Both the parent and child processes will continue execution from the point after fork().</p>

```
int main()
{
    pid_t pid;
    int x = 1;

    if((pid = fork()) < 0)
    {
        fprintf(stderr, "fork error: %d\n", strerror(errno));
        exit(0);
    }
    if(pid == 0)
    {
        printf("child : x=%d\n", ++x);
        exit(0);
    }
    
    printf("parent : x=%d\n", --x);
    exit(0);
}
```

- Can't predict execution order of parent and child

- Duplicate but separate address space

- <b>Shared open files</b>: stdout is the same to both parent and child

</br>

### Modeling fork with Process Graphs

</br>

<p>Forks can be kind of complex, especially if they're nested or you call them multiple times. Because we can't make any assumption about the ordering.</p>

<p>But a process graph is a useful tool for capturing the partial ordering of statements in a concurrent program: <b>Any topological sort of the graph corresponds to a feasible total ordering.</b></p>

<img width="1540" height="712" alt="QQ_1767426763111" src="https://github.com/user-attachments/assets/aae7f5bb-bf64-49ab-ae68-427f817ba71f" />

<p>Two consecutive forks:</p>

```
void fork2()
{
    printf("L0\n");
    fork();
    printf("L1\n");
    fork();
    printf("Bye\n");
}
```

<img width="834" height="614" alt="QQ_1767426956494" src="https://github.com/user-attachments/assets/fd5bb3e5-0307-44ce-882c-eecee87056c4" />

<p>Nested forks:</p>

```
void fork4()
{
    printf("L0\n");
    if(fork() != 0)
    {
        printf("L1\n");
        if(fork() != 0)
        {
            printf("L2\n");
        }
    }
    printf("Bye\n");
}
```

<img width="848" height="292" alt="QQ_1767427195577" src="https://github.com/user-attachments/assets/56bd9e14-9c13-464c-986b-cf9e6b0c42f1" />

</br>

### Reaping Child Processes

</br>

- When process terminates, it still consumes system resources

  - Examples: Exit status, various OS tables
 
- Called a "zombie"

  - Living corpse, half alive and half dead
 
<p>The reason it does this is that <b>the parent may want to know about the exit status of the child.</b> The system leaves it but doesn't remove it entirely. System keeps a little bit of <b>status associated with child in a form of the exit status or a OS table</b>. </p>

<b>Reaping: </b>

- Performed by parent on terminated child (using wait or waitpid)

- Parent is given exit status information

- Kernel then deletes zombie child process

<p>What if parent doesn't reap: </p>

- If any parent terminates without reaping a child, then the orphaned child will be reaped by <b>init</b> process (pid == 1)

- So, only need explicit reaping in long-running processes

  - e.g., shells and servers
 
<p>It's actually a form of memory leak.</p>

```
// forks.c
void fork7()
{
    if(fork() == 0)
    {
        printf("Terminating Child, PID = %d\n", getpid());
        exit(0);
    }
    else
    {
        printf("Running Parent, PID = %d\n", getpid());
        while(1)
            ;
    }
}
```

<p>This is a parent that never reaps the child that it created</p>

```
linux> ./forks 7 &
[1] 6639
Running Parent, PID = 6639
Terminating Child, PID = 6640
linux> ps
  PID TTY          TIME CMD
 6585 ttyp9    00:00:00 tcsh
 6639 ttyp9    00:00:03 forks
 6640 ttyp9    00:00:00 forks <defunct>
 6641 ttyp9    00:00:00 ps
linux> kill 6639
[1]    Terminated
linux> ps
  PID TTY          TIME CMD
 6585 ttyp9    00:00:00 tcsh
 6642 ttyp9    00:00:00 ps
```

<p>`defunct` indicates that it's a zombie</p>

<p>Now what happens if the child doesn't terminate.</p>

```
// forks.c
void fork8()
{
    if(fork() == 0)
    {
        printf("Running Child, PID = %d\n", getpid());
        while(1)
            ;
    }
    else
    {
        printf("Terminating Parent, PID = %d\n", getpid());
        exit(0);
    }
}
```

```
linux> ./forks 8
Terminating Parent, PID = 6675
Running Child, PID = 6676
linux> ps
  PID TTY          TIME CMD
 6585 ttyp9    00:00:00 tcsh
 6676 ttyp9    00:00:06 forks
 6677 ttyp9    00:00:00 ps
linux> kill 6676
linux> ps
  PID TTY          TIME CMD
 6585 ttyp9    00:00:00 tcsh
 6678 ttyp9    00:00:00 ps
```

<p>Child still active. Must kill child explicitly, or else will keep running indefinitely</p>

</br>

#### wait: Synchronizing with Children

</br>

<p><b>Parent reaps a child by calling the wait function.</b></p>

<p>int wait(int *child_status)</p>

- Suspends current process until one of its children terminates

- Return value is the pid of the child process that terminated

- If child_status != NULL, then the integer it points to will be set to a value that indicates reason the child terminated and the exit status:

  - Checked using macros defined in wait.h: WIFEXITED, WEXITSTATUS, WIFSIGNALED, WTERMSIG, WIFSTOPPED, WSTOPSIG, WIFCONTINUED
 
```
void fork9()
{
    int child_status;

    if(fork() == 0)
    {
        printf("HC: hello from child\n");
        exit(0);
    }
    else
    {
        printf("HP: hello from parent\n");
        wait(&child_status);
        printf("CT: child has terminated\n");
    }
    printf("Bye\n");
}
```

<img width="542" height="370" alt="QQ_1767451910744" src="https://github.com/user-attachments/assets/c117e9f3-bad0-4e5e-92a2-4b8bba6df988" />

<p>If multiple children completed, it will take in arbitrary order. We can use macros <b>WIFEXITED</b> and <b>WEXITSTATUS</b> to get information about exit status.</p>

```
void fork10()
{
    pid_t pid[N];
    int i, child_status;

    for(i = 0; i < N; i ++)
        if((pid[i] = fork()) == 0)
        {
            exit(100+i);
        }

    for(i = 0; i < N; i ++)
    {
        pid_t wpid = wait(&child_status);
        if(WIFEXITED(child_status))
            printf("Child %d terminated with exit status %d\n", wpid, WEXITSTATUS(child_status));
        else
            printf("Child %d terminates abnormally\n", wpid);
    }
    
}
```

<p>If WIFEXITED is false, it means that the child terminated for some other reason rather than exit</p>

</br>

#### waitpid

</br>

<p>pid_t waitpid(pid_t pid, int *status, int options)</p>

<p>Option: </p>

- 0: Default Blocks. and waits until the specified child process terminates (exit or is killed by a signal).

- WNOHANG: Non-blocking. If the child process has not yet terminated, then returns immediately to 0

- Various options (see textbook)

```
void fork11()
{
    pid_t pid[N];
    int i;
    int child_status;

    for(i = 0; i < N; i ++)
        if((pid[i] = fork()) == 0)
            exit(100 + i);

    for(i = N - 1; i >= 0; i --)
    {
        pid_t wpid = waitpid(pid[i], &child_status, 0);
        if(WIFEXITED(child_status))
            printf("Child %d terminated with exit status %d\n", wpid, WEXITSTATUS(child_status));
        else
            printf("Child %d terminates abnormally\n", wpid);
    }
}
```

</br>

### execve: Loading and Running Programs

</br>

<p><b>int execve(char *filename, char *argv[], char *envp[])</b></p>

<p>Loads and runs in the current process: </p>

- Executable file `filename`

  - Can be object file or script file beginning with `#!interpreter` (e.g., #!/bin/bash)

- Argument list `argv`

  - By convention argv[0] == filename

- Environment variable list `envp`

  - "name=value" strings (e.g., USER=droh)
 
  - getenv, putenv, printenv
 
- Completely overwrites virtual address space including code, data and stack

  - Retains PID, open files and signal context
 
- Called once and never returns <b>(except if there is an error)</b>

```
if (fork() == 0) {
    execve("/bin/ls", argv, envp);
    perror("execve");
    exit(1);
}
```
 
<p>If you want to write a shell script, the first line starts with a #! and then the path of some interpreter. That will execute bash, and then bash will read in the lines following and interpret them just like through you'd type them at the command line. As following: </p>

```
execve("/bin/bash",
       ["bash", "test.sh"],
       envp);
```

<p>Let's look at the <b>Structure of the stack when a new program starts</b>.</p>

<p>`execve` loads in new code and data, and creates a new stack frame. The stack frame that it creates has following.</p>

<img width="970" height="1126" alt="QQ_1767538704166" src="https://github.com/user-attachments/assets/6d6fb64c-a5cf-4827-b210-f3fcccf79119" />

<b>argv in %rsi. argc in %rdi.</b>

<p>Here is the situation right before the startup code calls main. The first function that executes is the `libc_start_main`. It has a stack frame. The future stack frame will be here at the top of stack.</p>

<p>There's some padding and then the argument list in argv is contained on the stack. So the argv is a list of pointers terminated by a null. Each one of these pointers points to an string that corresponds to an argument of the program you specified to run.</p>

<p>Environment list is also contained on the stack. His situation is the same as the above.</p>

<hr>

<p>execv Example: </p>

<p>We want to execute `/bin/ls -lt /usr/include` in child process using current environment: </p>

<img width="1326" height="534" alt="QQ_1767707939874" src="https://github.com/user-attachments/assets/4c9cea02-fae4-4d42-b8b4-8b37533ba70d" />

```
if((pid = fork()) == 0)
{
    if(execv(myargv[0], myargv, environ) < 0)
    {
        printf("%s: Command not found.\n", myargv[0]);
        exit(1);
    }
}
```

<p>The program we want to execute is always contained in the first element of argv.</p>

</br>

# Signals and Nonlocal Jumps

</br>

<p>Continue our study of ECF by looking at some higher level mechanisms known as Linux signals and C nonlocal jumps.</p>

</br>

## Signals

</br>

<p>There's one way to create process on Linux, that's using a fork call. But in fact all of the processes on the system actually form a hierarchy. And the very first process created when you boot the system up is the init process.</p>

<img width="1430" height="974" alt="QQ_1768644592242" src="https://github.com/user-attachments/assets/f94649b3-db4e-4de5-aaca-98ff7e624c4d" />

<p>All other processes are descendants of init process. When init process starts up it creates deamons which are long-running programs that provide services. Then it creates a so called login shells process which provide the command-line interface.</p>

```
int main()
{
    char cmdline[MAXLINE];

    while(1)
    {
        printf("> ");
        Fgets(cmdline, MAXLINE, stdin);

        if(feof(stdin))
            exit(0);

        eval(cmdline);
    }
}

void eval(char *cmdline)
{
    char *argv[MAXARGS];
    char buf[MAXLINE];
    int bg;
    pid_t pid;

    strcpy(buf, cmdline);
    bg = parseline(buf, argv);
    if(argv[0] == NULL)
        return;

    if(!builtin_command(argv))
    {
        if((pid = fork()) == 0)
        {
            if(execve(argv[0], argv, environ) < 0)
            {
                printf("%s: Command not found.\n", argv[0]);
                exit(0);
            }
        }

        // Whether to wait
        if(!bg)
        {
            int status;
            if(waitpid(pid, &status, 0) < 0)
                unix_error("waitfg: waitpid error");
        }
        else
            printf("%d %s", pid, cmdline);
    }
    return;
}
```

</br>

## Nonlocal jumps

</br>

<p></p>




































































































































































































































































































































































































































































































































































