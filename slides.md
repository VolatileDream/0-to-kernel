%title: From 0 to Kernel
%author: Gianni Gambetti (VolatileDream)
%date: 2014
%comments: A talk about building a kernel targeted at 2nd year CS students at UW.

`ker·nel`

> the central or most important part of something

-----------------------------------------------------------

## Overview

* About kernels
 * What
 * Why
* Kernel Style
* Writing your own kernel
 * Before Starting
 * IO
 * Scheduling
 * Context Switch
 * System Calls
* Next steps and more reading

-----------------------------------------------------------

## What is a kernel?

* What it does
<br>
 * Handles system boot from other software & hardware ( *magic* )
<br>
 * _Usually_ talks to hardware on behalf of system software
<br>
 * Sets up the system to run other software
<br>
 * Handles complex operations
  * Virtual Memory
  * Filesystem access
  * Networking
  * USB

-----------------------------------------------------------

## Why have a kernel?

* Time Sharing (aka multi-program systems)
 * Require threads & scheduling
<br>
* Abstract Hardware Access & Common Interfaces
 * Filesystem (SCSI, USB, NFS, etc)
 * Network (Ethernet, Wifi, etc)

-----------------------------------------------------------

## Kernel Style

This talk is heavily influenced by micro-kernel architecture

* Micro
 * Message passing
 * Interrupts exposed to non-kernel processes
 * Many small services make up kernel functionality
 * Service reloading (in theory)
* Macro
 * One monolith
 * Responsible for everything
 * Re-entrant kernel code

-----------------------------------------------------------

## Writing a Kernel

    void main(void){
        print("hello world\n");
    }

That looks pretty easy, right?
<br>

Now the bad news: it's not that easy.

-----------------------------------------------------------

## Writing a Kernel - Steps

* _*Read Documentation*_

-----------------------------------------------------------

## Writing a Kernel - Reading Documentation

> Size of documentation is inversly proportional to component size
* One week of reading
<br>
* *~50 for TS-7200*
<br>
* *~600 for Cirrus Logic System on a Chip*
<br>
* *~1.5k for ARMv4 Technical + Architecture Reference Manuals*
<br>
* *~3.5k for Intel® 64 and IA-32 Architectures Software Developer Manuals*

-----------------------------------------------------------

## Writing a Kernel - Steps

* Read Documentation
* _*Write IO*_

-----------------------------------------------------------

## Writing a Kernel - Writing IO

* Remember the documentation you read?
<br>
* This is hard: You don't get any output until it works
<br>
* But it's easy: 14 lines of code

   1)   volatile int \* UART_IO = (int*) 0x808C000;
   2)   volatile int \* UART_FLAGS = UART_IO + UART_FLAG_OFFSET;

   3)   char read(){
   4)       while( !(UART_FLAGS & UART_IN_BITS) ) {
   5)           /* Busy Waiting \*/
   6)       }
   7)       return (char)\*data;
   8)   }

   9)   void write(char c){
   A)       while( !(UART_FLAGS & UART_OUT_BITS) ) {
   B)           /* Busy Waiting \*/
   C)       }
   D)       \*data = c;
   E)   }

-----------------------------------------------------------

## Writing a Kernel - Writing IO

* Remember the documentation you read?
* This is hard: You don't get any output until it works
* But it's easy: 14 lines of code
 * Where did those constants come from?

   1)   volatile int \* UART_IO = (int*) _*0x808C000*_;
   2)   volatile int \* UART_FLAGS = UART_IO + _*UART_FLAG_OFFSET*_;
   4)       while( !(UART_FLAGS & _*UART_IN_BITS*_) ) {
   A)       while( !(UART_FLAGS & _*UART_OUT_BITS*_) ) {
<br>

* They're all documented _*somewhere*_...
<br>
 * It takes a few days to find all the magic constants

-----------------------------------------------------------

## Writing a Kernel

What do we have now?

<br>
    void main(void){
        char * out = "hello world\n";
        while( *out ){
            write( *out );
            out++;
        }
        read();
    }

<br>
Input and Output let you do everything.

-----------------------------------------------------------

## Writing a Kernel - Steps

* Read Documentation
* Write IO
* _*Scheduling*_

-----------------------------------------------------------

## Writing a Kernel - Scheduling

Recall:
<br>
 * Mathematics: Scheduling is *HARD* (NP Complete)
<br>
 * Engineering: KISS - Keep It Simple Stupid

-----------------------------------------------------------

## Writing a Kernel - Scheduling

* Defines structure for Context Switch
 * Required data: Program Counter, Stack Pointer

-----------------------------------------------------------

## Writing a Kernel - Scheduling

* Defines structure for Context Switch
* Defines priority setup for threads
 * One or more
<br>
 * Optimal: 2+ for system + user

-----------------------------------------------------------

## Writing a Kernel - Scheduling

* Defines structure for Context Switch
* Defines priority setup for threads
* Multitasking: co-operative or pre-emptive
<br>
 * Co-operative: threads give up CPU
<br>
 * Pre-emptive: interrupts bump threads off
<br>
 * Assumptions?

-----------------------------------------------------------

## Writing a Kernel - Scheduling

    struct proc {
        void * program_counter;
        void * stack_pointer;
        // other kernel state stuff
        //
        // scheduler data
        int priority; // optional
        proc * next; // next process linked list
    };

    // scheduler interface
    //
    // get the next process to run
    proc* scheduler_next();
    // add a process to the scheduler
    void  scheduler_add( proc* );

-----------------------------------------------------------

## Writing a Kernel - Steps

* Read Documentation
* Write IO
* Scheduling
* _*Context Switch*_

-----------------------------------------------------------

## Writing a Kernel - Context Switch

* What is the Context Switch?
<br>
 * It switches between running the kernel and user programs
<br>
 * A bunch of Assembly

-----------------------------------------------------------

## Writing a Kernel - Context Switch

* What is the Context Switch?
* Remember the documentation you read?
<br>
* Figure out the CPU modes/protection
<br>
 * These are fancy things that make life painful
<br>
 * But are required to implement more advanced features
  * Hardware Interrupts
  * Memory Protection/Virtual Memory
  * Handling illegal instruction execution

-----------------------------------------------------------

## Writing a Kernel - Context Switch

* What is the Context Switch?
* Remember the documentation you read?
* Figure out the CPU modes/protection
* Write some Assembly

-----------------------------------------------------------

## Writing a Kernel - Assembly

In pseudo-assembly:

    // Assembly file header stuff goes here.
    enter_kernel:
         get user stack pointer
         save user registers
         get kernel stack pointer
         save user stack pointer
         restore kernel registers
         jump into the kernel
    exit_kernel:
         save kernel registers
         get user stack pointer
         restore user registers
         jump into user program
    // Assembly file footer stuff goes here.

<br>
You're probably wondering why this is so hard.

-----------------------------------------------------------

## Writing a Kernel - Assembly

* You create the storage format for user data
 * Maintains process information (stack, program counter, etc.)
 * Assignment error checking
<br>
* If anything doesn't work exactly right everything breaks
 * And it invokes your context switch
<br>
* It can be subtly wrong and take ~1/2h to break
 * IP register blunder
<br>
* You can't unit test it

-----------------------------------------------------------

## Writing a Kernel - Context Switch

* What is the Context Switch?
* Remember the documentation you read?
* Figure out the CPU modes/protection
* Write some Assembly

* IT'S ALIVE!
 * In theory now it all works

-----------------------------------------------------------

## Writing a Kernel - Steps

* Read Documentation
* Write IO
* Scheduling
* Context Switch
* _*System Calls*_

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
<br>
 * User program talking to the kernel
<br>
 * Magic that user programs can't do
<br>
 * Examples
  * Process creation
  * Interrupt handling
  * Sleeping

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
* How does it work
<br>
 * Trigger an interrupt
<br>
  * Standardized by architecture
<br>
  * Special Assembly instructions
<br>
 * Enter the kernel
<br>
 * Kernel does magic
<br>
 * Exit the kernel

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
* How does it work
* Passing data
<br>
 * The kernel can do anything
<br>
 * It reads your data

-----------------------------------------------------------

## Writing a Kernel - Steps

* Read Documentation
* Write IO
* Scheduling
* Context Switch
* System Calls
* _*Now What?*_

-----------------------------------------------------------

## Writing a Kernel - Now What?

* eudyptula-challenge.org - linux kernel development intro
* wiki.osdev.org - walkthrough for your first kernel
* minix3.org - learning micro-kernel
* reddit.com/r/osdev - kernel development discussion

Questions?

