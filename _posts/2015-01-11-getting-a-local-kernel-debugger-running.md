---
layout: post
title: Getting a Local Kernel Debugger Running
date: 2015-01-11 19:12
image: /images/windbg.svg
tag: A brief walkthrough for WinDbg in local kernel mode
categories: [developing, kernel mode, operating systems, software, software engineering, windows internals]
---

There is a steep learning curve when getting familiar with Windows Internals. As a first step, it is a good idea to get a local kernel debugger working on your machine. This will allow you to follow along with whatever learning materials are at hand (e.g. Windows Internals). Here's a minimal path to get windbg.exe (pronounced "Wind Bag") up and running:

1. Install the latest [Debugging Tools for Windows](http://msdn.microsoft.com/en-us/windows/hardware/hh852365)
2. Navigate to the installation directory, e.g. `C:\Program Files\Debugging Tools for Windows (x64)`, in Explorer.
3. Right click on windbg.exe and click `Run as Administrator`
4. Create a folder for local symbols, e.g. `C:\LocalSymbols`
5. With windbg.exe open, click `File > Symbol File Path`
6. In Symbol File Path, `SRV*C:\LocalSymbols*http://msdl.microsoft.com/download/symbols`
7. Click `File > Kernel Debug`
8. Select the Local tab, press OK

If all is well, you should be good to go. Try, e.g., to print the interrupt descriptor table (IDT) with the command `!idt`

You should get output like:

```
lkd> !idt

Dumping IDT: fffff80000b95080

00: fffff80002e80940 nt!KiDivideErrorFault
01: fffff80002e80a40 nt!KiDebugTrapOrFault
02: fffff80002e80c00 nt!KiNmiInterrupt Stack = 0xFFFFF80000BA7000
03: fffff80002e80f80 nt!KiBreakpointTrap
04: fffff80002e81080 nt!KiOverflowTrap
05: fffff80002e81180 nt!KiBoundFault
06: fffff80002e81280 nt!KiInvalidOpcodeFault
07: fffff80002e814c0 nt!KiNpxNotAvailableFault
08: fffff80002e81580 nt!KiDoubleFaultAbort Stack = 0xFFFFF80000BA5000
09: fffff80002e81640 nt!KiNpxSegmentOverrunAbort
0a: fffff80002e81700 nt!KiInvalidTssFault
0b: fffff80002e817c0 nt!KiSegmentNotPresentFault
0c: fffff80002e81900 nt!KiStackFault
0d: fffff80002e81a40 nt!KiGeneralProtectionFault
0e: fffff80002e81b80 nt!KiPageFault
10: fffff80002e81f40 nt!KiFloatingErrorFault
11: fffff80002e820c0 nt!KiAlignmentFault
12: fffff80002e821c0 nt!KiMcheckAbort Stack = 0xFFFFF80000BA9000
13: fffff80002e82540 nt!KiXmmException
1f: fffff80002e767d0 nt!KiApcInterrupt
2c: fffff80002e82700 nt!KiRaiseAssertion
2d: fffff80002e82800 nt!KiDebugServiceTrap
2f: fffff80002ece950 nt!KiDpcInterrupt
```
