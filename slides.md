%title: From 0 to Kernel and Some Real-Time stuff
%author: Gianni Gambetti (VolatileDream)
%date: 2014
%comments: A talk about kernels targeted at 2nd year CS students at UW.


`ker·nel`

> the central or most important part of something

-----------------------------------------------------------

## Overview

* about kernels
* micro vs macro
* writing your own kernel
* bits about real-time
* next steps (or continued reading)

-----------------------------------------------------------

## What is a kernel?

* what it does
  - handles system boot (from other software/hardware)
  - sets up the system to run other software
  - talks to hardware on behalf of system software
* where it lives
  - a binary image on somewhere on your filesystem

-----------------------------------------------------------

## When and Where is the kernel

* When
* Where

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

-----------------------------------------------------------

## Writing a Kernel

Your first kernel, let's start simple.

    void main(void){
        print("hello world\n");
    }

Wasn't that easy?

-----------------------------------------------------------

