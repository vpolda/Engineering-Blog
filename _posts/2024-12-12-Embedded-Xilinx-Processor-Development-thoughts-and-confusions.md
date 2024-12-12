---
layout: post
title:  "Embedded Xilinx Processor Development: Thoughts and Confusions"
date:   2024-12-12T11:00:00-05:00
author: Victoria Polda
categories: Software
---

I have been working with AXI Video Direct Memory Access (VDMA or video DMA) for my RTL Edge detection and live video processing project. The problem I am running into, is understanding the C code and what is happening. 

For reference, I am solid in C and CPP, and have plenty of school experience working with it. More advanced application elude me but I am comfortable for the most part. In this post, I want to go through some of my complaints with the resources out there as well as documenting some of my issues and covering what I wish I had seen.

I can't seem to grasp the intricacies of the built in Xilinx drivers and code. Tutorials out there simply give you the TCL script to generate the program or only hand you the example code. I have yet to find some that explain what is going on beside a few rare gems like this blog by Lauri Vosandi: https://lauri.võsandi.com/posts.html (thank you!). Me and my googling skills are probably to blame here. So I have been going through it with ChatGPT to try and understand what is actually happening. Using Xilinx code is in theory optional, difficult, but optional. And many of the examples out there show some of the different ways code can be developed or integrated with Xilinx's. These leads to a mess for me. Trying to sort through to find what is NEEDED for any driver, instead of mindlessly copying their code. 

For example, why does examples of the VDMA either use the XScuGIC dedicated interrupts or software based callbacks or a combo for interrupts from the VDMA IP module? 
The answer lies in the Xilinx code (and probably what people want to do or something else I'm missing). 

In depth looks
XScuGIC (Xilinx Self-Contained General Interrupt Controller): This proprietary hardware from Xilinx based driver receives interrupts from the PL and then calls functions in the PS. 

The following shows what steps are done to get this driver racing:
Interrupt Controller Initialization: 
 - uses XScuGic_CfgInitialize() function
 - provides a reference to the interrupt controller instance and configuration settings based on the hardware address in the xparameters.h file

Interrupt Sources Configuration: 
 - Uses XScuGIC_Connect() function to connect PL to PS interrupts to a function contained in the program
 - This has to happen for each channel, like read and write
 - If using the Xilinx included XAxiVdma_WriteIntrHandler, this takes care of connecting your ISR and other back side work. You can simply add your code the the callback functions. These are not software based ones! That was my confusion.

Enable Interrupts: 
 - Uses the XScuGIC_Enable() function
 - This allows the interrupt controller to start receiving and handling interrupts from the configured sources

Exception Handling: In some cases, you need to configure an exception handler for interrupts that do not have a specific ISR. This ensures that no interrupts are left unhandled. 
It follows three steps:
  -  Xil_ExceptionInit()
  -	Xil_ExceptionRegisterHandler()
  -	Xil_ExceptionEnable()
