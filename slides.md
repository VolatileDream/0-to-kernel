%title: From 0 to Kernel and Some Other Stuff
%author: Gianni Gambetti (VolatileDream)
%date: 2014
%comments: A talk about building a kernel targeted at 2nd year CS students at UW.


`ker·nel`

> the central or most important part of something

-----------------------------------------------------------

## Overview

* about kernels
 * What
 * Where
 * Why
* micro vs macro
* writing your own kernel
 * Before Starting
 * IO
 * Scheduling
 * Context Switch
 * System Calls
* next steps (or continued reading)

-----------------------------------------------------------

## What is a kernel?

* what it does
 * handles system boot (from other software/hardware)
 * *This is magic*
 * sets up the system to run other software
 * _usually_ talks to hardware on behalf of system software
* where it lives
 * a binary image on somewhere on your filesystem

-----------------------------------------------------------

## When/Where is the kernel

* When
 * The kernel is always running
 * Some exceptions
* Where
 * Beneath the rest of the Operating System
 * Talking to the hardware
 * All your base are belong to it

-----------------------------------------------------------

## Why?

* multi user systems
  - pre-emptive scheduling
* indirect hardware access
  - gpu/hard disk type doesn't matter

-----------------------------------------------------------

## Micro vs Macro

-----------------------------------------------------------
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

* Read Documentation

-----------------------------------------------------------

## Writing a Kernel - Reading Documentation

* One week of light reading
* Size of documentation is inversly proportional to component size
* *~50 for TS-7200*
* *~600 for Cirrus Logic System on a Chip*
* *~1.5k for ARMv4 Technical + Architecture Reference Manuals*
* *~3.5k for Intel® 64 and IA-32 Architectures Software Developer Manuals*

-----------------------------------------------------------

## Writing a Kernel - Steps

* Read Documentation
* Write IO

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
