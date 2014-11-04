%title: From 0 to Kernel and Some Other Stuff
%author: Gianni Gambetti (VolatileDream)
%date: 2014
%comments: A talk about building a kernel targeted at 2nd year CS students at UW.


`ker·nel`

> the central or most important part of something

-----------------------------------------------------------

## Overview

* About kernels
 * What
 * Where
 * Why
* Kernel Style
* Writing your own kernel
 * Before Starting
 * IO
 * Context Switch
 * Scheduling
 * System Calls
* Next steps and more reading

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

## When/Where is the kernel?

* The kernel is (almost) always running
* Magically hidden in RAM

-----------------------------------------------------------

## Why have a kernel?

* Time Sharing (aka multi-program systems)
 * Require threads & scheduling
* Indirect hardware access
 * GPU type doesn't matter (OpenGL interface)
 * Networking/filesystem abstraction (read/write)

-----------------------------------------------------------

## Kernel Style

This talk is heavily influenced by micro-kernel architecture

* Micro
 * Many small services
 * Message passing
* Macro
 * One monolith
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
* One week of light reading
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
 * Probably not...
 * It took a few days to find all the magic constants

-----------------------------------------------------------

## Writing a Kernel - Context Switch

* What is the Context Switch?
* Remember the documentation you read?
 * Probably not...
 * It took a few days to find all the magic constants
 * After you check it into version control, you go out drinking

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

It is hard because:
* You create the storage format for user data
 * I've seen other people's assignments...
 * I don't trust you.
* If anything doesn't work exactly right everything breaks
 * And it invokes your context switch
* It can be subtly wrong and take ~1/2h to break
 * IP register blunder
