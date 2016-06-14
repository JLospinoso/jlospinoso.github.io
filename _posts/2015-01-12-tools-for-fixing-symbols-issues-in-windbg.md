---
layout: post
title: Tools for fixing symbols issues in WinDbg
date: 2015-01-12 03:08
image: /images/windbg.gif
tag: Symbols can give some trouble when WinDbg is first installed
categories: [developing, kernel mode, operating systems, software, software engineering, windows internals]
---
Before worrying about setting your environment variables, e.g. `_NT_SYMBOL_PATH`, get the Windbg session working.

Try:

```
lkd> .symfix
```

This will set your symbol search path to the Microsoft msdl symbol server. Next, reload your symbols:

```
lkd> .reload /f
```

This will force the debugger to immediately reload all symbols. This should fix the issue. If it does not, try

```
lkd> !sym noisy
```

This will turn on your symbol prompts. Try again running `.symfix`, looking for clues in the output.

**Once your symbols are fixed**, you can fix your environment variables. First, ensure that you have created a symbols cache, e.g. `C:\Symbols`. Add it to `.symfix` via the command:

```
lkd> .symfix +C:\Symbols
```

Ensure that symbols still work (do a `.reload`), then have a look at your `.sympath`:

```
lkd> .sympath

Symbol search path is: symsrv*symsrv.dll*C:\Symbols*http://msdl.microsoft.com/download/symbols
Expanded Symbol search path is: symsrv*symsrv.dll*c:\windowssymbols*http://msdl.microsoft.com/download/symbols
************* Symbol Path validation summary **************
Response Time (ms) Location
Deferred symsrv*symsrv.dll*C:\WindowsSymbols*http://msdl.microsoft.com/download/symbols
```

Now set your `_NT_SYMBOL_PATH` using the output above.

1. Right click on My Computer
2. Properties
3. Advanced
4. Environment Variables
5. New (User variables)
6. Variable name = `_NT_SYMBOL_PATH`
7. Variable value:

	`symsrv*symsrv.dll*C:\WindowsSymbols*http://msdl.microsoft.com/download/symbols`

Close Windbg and open a new session. Ensure that symbols are fixed. If not, examine your  `.sympath` and ensure it does not differ from the settings that worked in your previous session.
