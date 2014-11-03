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

Your first kernel, let's start simple.

* Read Documentation

-----------------------------------------------------------

## Writing a Kernel

Your first kernel, let's start simple.

* Read Documentation
 * One week of light reading
 * Size of documentation is inversly proportional to component size
 * *~50 for TS-7200*
 * *~600 for Cirrus Logic System on a Chip*
 * *~1.5k for ARMv4 Technical + Architecture Reference Manuals*
 * *~3.5k for Intel® 64 and IA-32 Architectures Software Developer Manuals*

-----------------------------------------------------------

## Writing a Kernel

Your first kernel, let's start simple.

* Read Documentation
 * One week of light reading
 * Size of documentation is inversly proportional to component size
 * *~50 for TS-7200*
 * *~600 for Cirrus Logic System on a Chip*
 * *~1.5k for ARMv4 Technical + Architecture Reference Manuals*
 * *~3.5k for Intel® 64 and IA-32 Architectures Software Developer Manuals*
 * But proportional to the difficulty of the software being written

-----------------------------------------------------------

## Writing a Kernel

* Read Documentation
* Write IO

-----------------------------------------------------------

## Writing a Kernel

* Read Documentation
* Write IO
 * This is hard: You don't get any output until it works
 * Maybe 30 lines of code

-----------------------------------------------------------

## Writing a Kernel

Your first kernel, let's start simple.

    void main(void){
        print("hello world\n");
    }

Wasn't that easy?

-----------------------------------------------------------

