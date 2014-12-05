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
 * Handles system boot from other software & hardware ( *magic* )

-----------------------------------------------------------

## What is a kernel?

* What it does
 * Handles system boot from other software & hardware ( *magic* )
 * _Usually_ talks to hardware on behalf of system software

-----------------------------------------------------------

## What is a kernel?

* What it does
 * Handles system boot from other software & hardware ( *magic* )
 * _Usually_ talks to hardware on behalf of system software
 * Sets up the system to run other software

-----------------------------------------------------------

## What is a kernel?

* What it does
 * Handles system boot from other software & hardware ( *magic* )
 * _Usually_ talks to hardware on behalf of system software
 * Sets up the system to run other software
 * Handles complex operations
  * Virtual Memory
  * Filesystem access
  * Networking
  * USB

-----------------------------------------------------------

## Why have a kernel?

* Time Sharing (aka multi-program systems)
 * Require threads & scheduling

-----------------------------------------------------------

## Why have a kernel?

* Time Sharing (aka multi-program systems)
 * Require threads & scheduling
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

-----------------------------------------------------------

## Writing a Kernel

    void main(void){
        print("hello world\n");
    }

That looks pretty easy, right?

Now the bad news: it's not that easy.

-----------------------------------------------------------

## Writing a Kernel - Steps

* _*Read Documentation*_

-----------------------------------------------------------

## Writing a Kernel - Reading Documentation

> Size of documentation is inversly proportional to component size
* One week of reading
* *~50 for TS-7200*
* *~600 for Cirrus Logic System on a Chip*
* *~1.5k for ARMv4 Technical + Architecture Reference Manuals*
* *~3.5k for Intel® 64 and IA-32 Architectures Software Developer Manuals*

-----------------------------------------------------------

## Writing a Kernel - Steps

* Read Documentation
* _*Write IO*_

-----------------------------------------------------------

## Writing a Kernel - Writing IO

* Remember the documentation you read?

-----------------------------------------------------------

## Writing a Kernel - Writing IO

* Remember the documentation you read?
* This is hard: You don't get any output until it works

-----------------------------------------------------------

## Writing a Kernel - Writing IO

* Remember the documentation you read?
* This is hard: You don't get any output until it works
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

* They're all documented _*somewhere*_...

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

* They're all documented _*somewhere*_...
 * It takes a few days to find all the magic constants

-----------------------------------------------------------

## Writing a Kernel

What do we have now?

-----------------------------------------------------------

## Writing a Kernel

    void main(void){
        char * out = "hello world\n";
        while( *out ){
            write( *out );
            out++;
        }
        read();
    }

-----------------------------------------------------------

## Writing a Kernel

    void main(void){
        char * out = "hello world\n";
        while( *out ){
            write( *out );
            out++;
        }
        read();
    }

Input and Output let you do everything.

-----------------------------------------------------------

## Writing a Kernel - Steps

* Read Documentation
* Write IO
* _*Scheduling*_

-----------------------------------------------------------

## Writing a Kernel - Scheduling

Recall:

-----------------------------------------------------------

## Writing a Kernel - Scheduling

Recall:
 * Mathematics: Scheduling is *HARD* (NP Complete)

-----------------------------------------------------------

## Writing a Kernel - Scheduling

Recall:
 * Mathematics: Scheduling is *HARD* (NP Complete)
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

-----------------------------------------------------------

## Writing a Kernel - Scheduling

* Defines structure for Context Switch
* Defines priority setup for threads
 * One or more
 * Optimal: 2+ for system + user

-----------------------------------------------------------

## Writing a Kernel - Scheduling

* Defines structure for Context Switch
* Defines priority setup for threads
* Multitasking: co-operative or pre-emptive

-----------------------------------------------------------

## Writing a Kernel - Scheduling

* Defines structure for Context Switch
* Defines priority setup for threads
* Multitasking: co-operative or pre-emptive
 * Co-operative: threads give up CPU

-----------------------------------------------------------

## Writing a Kernel - Scheduling

* Defines structure for Context Switch
* Defines priority setup for threads
* Multitasking: co-operative or pre-emptive
 * Co-operative: threads give up CPU
 * Pre-emptive: interrupts bump threads off

-----------------------------------------------------------

## Writing a Kernel - Scheduling

* Defines structure for Context Switch
* Defines priority setup for threads
* Multitasking: co-operative or pre-emptive
 * Co-operative: threads give up CPU
 * Pre-emptive: interrupts bump threads off
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

-----------------------------------------------------------

## Writing a Kernel - Context Switch

* What is the Context Switch?
 * Switch between running the kernel and user programs

-----------------------------------------------------------

## Writing a Kernel - Context Switch

* What is the Context Switch?
 * Switch between running the kernel and user programs
 * A bunch of Assembly

-----------------------------------------------------------

## Writing a Kernel - Context Switch

* What is the Context Switch?
* Remember the documentation you read?

-----------------------------------------------------------

## Writing a Kernel - Context Switch

* What is the Context Switch?
* Remember the documentation you read?

-----------------------------------------------------------

## Writing a Kernel - Context Switch

* What is the Context Switch?
* Remember the documentation you read?
* Figure out the CPU modes/protection

-----------------------------------------------------------

## Writing a Kernel - Context Switch

* What is the Context Switch?
* Remember the documentation you read?
* Figure out the CPU modes/protection
 * These are fancy things that make life painful

-----------------------------------------------------------

## Writing a Kernel - Context Switch

* What is the Context Switch?
* Remember the documentation you read?
* Figure out the CPU modes/protection
 * These are fancy things that make life painful
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

You're probably wondering why this is so hard.

-----------------------------------------------------------

## Writing a Kernel - Assembly

* You create the storage format for user data
 * Maintains process information (stack, program counter, etc.)
 * Assignment error checking

-----------------------------------------------------------

## Writing a Kernel - Assembly

* You create the storage format for user data
 * Maintains process information (stack, program counter, etc.)
 * Assignment error checking
* If anything doesn't work exactly right everything breaks
 * And it invokes your context switch

-----------------------------------------------------------

## Writing a Kernel - Assembly

* You create the storage format for user data
 * Maintains process information (stack, program counter, etc.)
 * Assignment error checking
* If anything doesn't work exactly right everything breaks
 * And it invokes your context switch
* It can be subtly wrong and take ~1/2h to break
 * IP register blunder

-----------------------------------------------------------

## Writing a Kernel - Assembly

* You create the storage format for user data
 * Maintains process information (stack, program counter, etc.)
 * Assignment error checking
* If anything doesn't work exactly right everything breaks
 * And it invokes your context switch
* It can be subtly wrong and take ~1/2h to break
 * IP register blunder
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

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
 * User program talking to the kernel

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
 * User program talking to the kernel
 * Magic that user programs can't do

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
 * User program talking to the kernel
 * Magic that user programs can't do
 * Examples
  * Process creation
  * Interrupt handling
  * Sleeping

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
* How does it work

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
* How does it work
 * Trigger an interrupt

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
* How does it work
 * Trigger an interrupt
  * Standardized by architecture

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
* How does it work
 * Trigger an interrupt
  * Standardized by architecture
  * Special Assembly instructions

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
* How does it work
 * Trigger an interrupt
  * Standardized by architecture
  * Special Assembly instructions
 * Enter the kernel

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
* How does it work
 * Trigger an interrupt
  * Standardized by architecture
  * Special Assembly instructions
 * Enter the kernel
 * Kernel does magic

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
* How does it work
 * Trigger an interrupt
  * Standardized by architecture
  * Special Assembly instructions
 * Enter the kernel
 * Kernel does magic
 * Exit the kernel

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
* How does it work
* Passing data

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
* How does it work
* Passing data
 * The kernel can do anything

-----------------------------------------------------------

## Writing a Kernel - System Calls

* What's a system call?
* How does it work
* Passing data
 * The kernel can do anything
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

