---
layout: post
title: Getting Started with Reverse Engineering
date: 2015-03-03 21:00
image: /images/2015_03_03_1.JPG
tag: Basic static analysis with strings, disassembly, and IDA
categories: [developing, software, software engineering, reverse engineering, assembly]
---
[1]: http://undocumented.ntinternals.net/
[2]: http://blog.kaspersky.com/billion-dollar-apt-carbanak/
[3]: http://www.metasploit.com/
[4]: https://www.endgame.com/blog/defcon-capture-the-flag-qualification-challenge-1.html
[5]: http://filebin.ca/1tq2s3LGLGcV/reversing-demo.exe
[6]: https://www.hex-rays.com/products/ida/support/download.shtml
[7]: https://technet.microsoft.com/en-us/sysinternals/bb842062
[8]: https://msdn.microsoft.com/en-us/library/yd4f8bd1%28vs.71%29.aspx
[9]: http://support.microsoft.com/kb/177429
[10]: https://www.visualstudio.com/en-us/downloads/download-visual-studio-vs.aspx
[11]: http://filebin.ca/1tt5rHfH9IeI/ConsoleApplication1.zip
[12]: http://filebin.ca/1tt64OgFqLGk/ConsoleApplication1.pdb
[13]: https://msdn.microsoft.com/en-us/magazine/cc301805.aspx

The ability to reverse engineer binaries is extremely important in many settings. Whether [analyzing malware][2] (or [writing malware][3]...), delving into [undocumented APIs][1], or even just [for fun][4], you will not have the source available. Any kind of thorough reversing effort will invariably involve staring at lots of assembly (or perhaps Java bytecode/.NET IL for managed code).

There are three kinds of reverse engineering analysis:

1. *Static analysis* involves analysis of the contents of the binary file. This entails determining structure of the executable portions (typically manifested in lots of assembly) and printing out readable portions for hints about the program's purpose.
2. *Dynamic analysis* involves executing the binary (perhaps attaching a debugger) to ascertain what the binary's intended purpose is, how it does it, etc.
3. *Hybrid analysis* is a mixture of the two. Iterating between static analysis of a codepath, followed by a detailed debugging (or vice versa!) can often lend insights greater than could be obtained by either alone.

# A Simple Binary
We'll be using [this simple binary][5] (MD5 checksum is `255134D98BC4A524A1777D16FF8C2642`). I've put the source at the end of the file (in the *Source* section). 

# Strings
One basic item of static analysis we can perform on a binary is to "run strings" on it. The Strings program, available for free as part of the [Sysinternals][7] suite of tools, dumps out all of the--you guessed it--strings that appear in the binary. Install Sysinternals and put it into your path. Navigate to the directory of the binary, and issue the command:

	> strings --help

The following helpful prompt is given in response:

	Usage: strings [option(s)] [file(s)]
	 Display printable strings in [file(s)] (stdin by default)
	 The options are:
	  -a - --all                Scan the entire file, not just the data section
	  -f --print-file-name      Print the name of the file before each string
	  -n --bytes=[number]       Locate & print any NUL-terminated sequence of at
	  -<number>                   least [number] characters (default 4).
	  -t --radix={o,d,x}        Print the location of the string in base 8, 10 or 16
	  -o                        An alias for --radix=o
	  -T --target=<BFDNAME>     Specify the binary file format
	  -e --encoding={s,S,b,l,B,L} Select character size and endianness:
				    s = 7-bit, S = 8-bit, {b,l} = 16-bit, {B,L} = 32-bit
	  @<file>                   Read options from <file>
	  -h --help                 Display this information
	  -v -V --version           Print the program's version number
	strings: supported targets: ...
	Report bugs to <http://www.sourceware.org/bugzilla/>
		
Let's give the defaults a try:

	> strings reversing-demo.exe

Cutting out some of the unreadable results, we're left with these:

	%c%c%c%c
	C:\Users\jalospinoso\ReversingDemo\ConsoleApplication1\Release\ConsoleApplication1.pdb
	printf
	MSVCR120.dll
	_XcptFilter
	_amsg_exit
	__getmainargs
	__set_app_type
	exit
	_exit
	_cexit
	_configthreadlocale
	__setusermatherr
	_initterm_e
	_initterm
	__initenv
	_fmode
	_commode
	_crt_debugger_hook
	__crtUnhandledException
	__crtTerminateProcess
	?terminate@@YAXXZ
	__crtSetUnhandledExceptionFilter
	_lock
	_unlock
	_calloc_crt
	__dllonexit
	_onexit
	_invoke_watson
	_controlfp_s
	_except_handler4_common
	EncodePointer
	IsDebuggerPresent
	IsProcessorFeaturePresent
	QueryPerformanceCounter
	GetCurrentProcessId
	GetCurrentThreadId
	GetSystemTimeAsFileTime
	DecodePointer
	KERNEL32.dll
	<?xml version='1.0' encoding='UTF-8' standalone='yes'?>
	<assembly xmlns='urn:schemas-microsoft-com:asm.v1' manifestVersion='1.0'>
	  <trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">
	    <security>
	      <requestedPrivileges>
		<requestedExecutionLevel level='asInvoker' uiAccess='false' />
	      </requestedPrivileges>
	    </security>
	  </trustInfo>
	</assembly>
		
We've got some embedded information here about some `.pdb` [file][8]. This file, if it were available, would contain formation (like function prototypes) that would make debugging much easier. The binary probably imports the C runtime (which we can tell from `MSVCR120.dll`), and it was probably compiled with Visual Studio 2013--information we can glean off the `120` version number in the .dll. It looks like the binary calls `printf`, but we can't really infer much more.

# Dumpbin

We can confirm our suspicions about what functions the file calls with the [Dumpbin utility][9], which is available as part of Microsoft Visual C++ (you can obtain this e.g. by installing [Visual Studio 2013][10]). Once you have it installed (one way or another), you can put it on your PATH. Mine could be found here:

	C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\dumpbin.exe

Dumpbin is a powerful utility. We'll use it to view the functions that our binary depends on:

	> dumpbin /IMPORTS reversing-demo.exe

	Here are the results:

	Microsoft (R) COFF/PE Dumper Version 12.00.31101.0
	Copyright (C) Microsoft Corporation.  All rights reserved.


	Dump of file reversing-demo.exe

	File Type: EXECUTABLE IMAGE

	  Section contains the following imports:

	    MSVCR120.dll
			402024 Import Address Table
			4022B4 Import Name Table
			     0 time date stamp
			     0 Index of first forwarder reference

			  1F4 __setusermatherr
			  30D _initterm_e
			  30C _initterm
			  1B7 __initenv
			  2A2 _fmode
			  23F _commode
			  250 _crt_debugger_hook
			  1AC __crtUnhandledException
			  1AB __crtTerminateProcess
			  240 _configthreadlocale
			  135 ?terminate@@YAXXZ
			  1A9 __crtSetUnhandledExceptionFilter
			  394 _lock
			  504 _unlock
			  22E _calloc_crt
			  1AE __dllonexit
			  43A _onexit
			  314 _invoke_watson
			  243 _controlfp_s
			  27A _except_handler4_common
			  22F _cexit
			  283 _exit
			  64E exit
			  1F2 __set_app_type
			  1B6 __getmainargs
			  217 _amsg_exit
			  16B _XcptFilter
			  6FD printf

	    KERNEL32.dll
			402000 Import Address Table
			402290 Import Name Table
			     0 time date stamp
			     0 Index of first forwarder reference

			  2D6 GetSystemTimeAsFileTime
			  20E GetCurrentThreadId
			  20A GetCurrentProcessId
			  42D QueryPerformanceCounter
			  36D IsProcessorFeaturePresent
			  367 IsDebuggerPresent
			  121 EncodePointer
			   FE DecodePointer

	  Summary

		1000 .data
		1000 .rdata
		1000 .reloc
		1000 .rsrc
		1000 .text
		
After staring at enough Dumpbin dumps, it might be possible to tease out some unusual functions in here, but it's tough to say for sure. One thing that does stick out is the `printf` import.Maybe we can try running the binary to see what happens (although, I totally understand if you wouldn't want to run it without having comiled it yourself!)

	> reversing-demo.exe

Nothing seems to have happened We still really don't know what our binary is doing.

# IDA Free
We'll be using the free version of the *interactive disassembler* IDA 5.0 Freeware available [here][6]. It turns out that disassembling a binary is pretty complicated, and IDA is widely regarded as the best tool for doing it. After installing the free version, fire it up.

1. Click on "New : Disassemble a new file"
2. Click "PE Executable" and click "OK"
3. Navigate to "reversing-demo.exe" and Open it.
4. Let's use the default options in the wizard. Continue clicking "Next >" until complete.
5. As we guessed from the `.pdb` file in Strings, the binary was linked with debug information. Helpfully, a pop-up box confirms this and asks if we have the PDB available. I've put the PDB into the links section below, but try reversing without this information first. Click "No" when asked to link in the PDB file.

IDA will do a whole bunch of work at this point trying to disassemble your file.

Let's do a very quick orientation to IDA. First thing to notice is the slider:

![IDA Slider]({{site.url}}/images/2015_03_03_1/slider.jpg)

This slider allows you to click into a region of the binary and investigate its contents. For example,
the pink region contains all of the `.idata` which is [PE Format][13] speak for all the imports. You can see function definitions that should correspond with our Dumpbin analysis from earlier (side project: investigate the "Exports" and "Imports" tabs to see if they correspond with the output we found).

Clicking into a blue region navigates us to a `.text` section that contains all of our executable code (assembly):

![IDA Slider]({{site.url}}/images/2015_03_03_1/ida_view_space.jpg)

You can scroll through this disassembly and, in theory, figure out what's going on from here. Fortunately, IDA has a graph version of this "IDA View" which can be accessed by pressing the `Space` bar:

![IDA Slider]({{site.url}}/images/2015_03_03_1/ida_view.jpg)

This view is perhaps what IDA is best known for: each block represents a chunk of code, and the lines represent jumps between the code blocks (both conditional and unconditional). This is much, much easier to follow than scrolling through pages and pages of disassembly.

The final window we'll explore is the strings view:

![IDA Slider]({{site.url}}/images/2015_03_03_1/strings.jpg)

You'll notice that the output corresponds with what we got from running Strings earlier.

There are a great many ways to start analysis of a binary in IDA, but the most important thing to keep in mind is what your goal is. As you'll see, this binary is extremely simple--but it would take quite a bit of time to step through the assembly code line-by-line to try to figure out what's going on.

Well, we ran the binary and nothing happened, but there's a `printf` statement that showed up in both Strings and Dumpbin. Take a look at the Strings window and identify the `printf` line. It should appear at `.rdata:0040232A`. Let's have a look at the `Imports` tab:

![IDA Slider]({{site.url}}/images/2015_03_03_1/imports.jpg)

I've highlighted the line where `printf` is imported from `MSVCR120.dll`. Let's click on this entry, which brings us into the end of the `.idata` section:

	.idata:00402090 ; int printf(const char *,...)
	.idata:00402090                 extrn printf:dword      ; DATA XREF: sub_401000+4Dr

The IDA hotkey `x` is immensely useful here. We'd like to find out where else in the binary this import is referred to. After clicking on `printf`, press `x`:

![IDA Slider]({{site.url}}/images/2015_03_03_1/xrefs.jpg)

We can see here that `printf` is referred to one time in a subroutine (`sub_401000`). Let's click on it and enter the graph view around this call.

![IDA Slider]({{site.url}}/images/2015_03_03_1/printf-routine.jpg)

# CONTINUE HERE

# Spoiler alert!
Try not to look at the below sections before trying to reverse the binary yourself!

### Source
	#include "stdio.h"

	int main(int argc, char *argv[]) {
		volatile a = 0x0a;
		volatile b = 0x27;
		volatile c = 0x3b;
		volatile d = 0x63;
		if (argc == 2) {
			a ^= 0x42;
			b ^= 0x42;
			c ^= 0x42;
			d ^= 0x42;
			printf("%c%c%c%c\n", a, b, c, d);
		}
		return 0;
	}

### Links to Binary/PDB
* [Source][12] (MD5: CB2EE7B867334D1AD8844E7F03EB89B9)
* [PDB][11] (MD5: AFC2ADB7ED0E2C01AE1EAEDB5FA78B80)
* [Binary][5] (MD5: 255134D98BC4A524A1777D16FF8C2642)
