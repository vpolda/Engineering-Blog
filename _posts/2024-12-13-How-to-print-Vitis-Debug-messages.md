---
layout: post
title:  "How to print Vitis Debug messages in Baremetal applications"
date:   2024-12-13T13:00:00-05:00
author: Victoria Polda
categories: Software
---

How to have statements like:
```c
xdbg_printf(XDBG_DEBUG_ERROR, "Read channel reset failed %x\n\r", (unsigned int)XAxiVdma_ChannelGetStatus(RdChannel));
```
print to your active debug programming port.

Is it through "proc_extra_compiler_flags" adding -DDEBUG or including XDEBUG?

If proc flags:
How to enable: add -DDEBUG to the "proc_extra_compiler_flags"

What it's doing:
Appends the flag "DDEBUG" to the compiler which tells it to compile those relevant commands, like xdbg_printf, into assembly and then later binary machine code. 
It is recommended to use an:
```c
#ifdef DEBUG
    xdbg_printf("Debugging is enabled!\n");
#endif
```
This is done to output the debug statements only if debug is enabled. The preprocessor handles these statements.