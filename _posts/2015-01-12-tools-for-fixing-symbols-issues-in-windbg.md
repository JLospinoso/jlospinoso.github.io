---
layout: post
title: Tools for fixing symbols issues in Windbg
date: 2015-01-12 03:08
author: jalospinoso
comments: true
categories: [developing, kernel mode, operating systems, software, software engineering, windows internals]
---
Before worrying about setting your environment variables, e.g. _NT_SYMBOL_PATH, get the Windbg session working.

Try:
<code>
lkd&gt; .symfix
</code>

This will set your symbol search path to the microsoft msdl symbol server. Next, reload your symbols:
<code> lkd&gt; .reload /f
</code>

This will force the debugger to immediately reload all symbols. This should fix the issue. If it does not, try
<code> lkd&gt; !sym noisy
</code>

This will turn on your symbol prompts. Try again running <code>.symfix</code>, looking for clues in the output.

<strong>Once your symbols are fixed</strong>, you can fix your environment variables. First, ensure that you have created a symbols cache, e.g. <code>C:\Symbols</code>. Add it to <code>.symfix</code> via
Try:
<code>lkd&gt; .symfix +C:\Symbols
</code>

Ensure that symbols still work (do a <code>.reload</code>), then have a look at your <code>.sympath</code>:
Try:
<code>lkd&gt; .sympath
Symbol search path is: symsrv*symsrv.dll*C:\Symbols*http://msdl.microsoft.com/download/symbols
Expanded Symbol search path is: symsrv*symsrv.dll*c:\windowssymbols*http://msdl.microsoft.com/download/symbols
************* Symbol Path validation summary **************
Response Time (ms) Location
Deferred symsrv*symsrv.dll*C:\WindowsSymbols*http://msdl.microsoft.com/download/symbols</code>

Now set your <code>_NT_SYMBOL_PATH</code> using the output above.
<ol>
	<li>Right click on My Computer.</li>
	<li>Properties</li>
	<li>Advanced</li>
	<li>Environment Variables</li>
	<li>New (User variables)</li>
	<li>Variable name = <code>_NT_SYMBOL_PATH </code></li>
	<li>Variable value <code>symsrv*symsrv.dll*C:\WindowsSymbols*http://msdl.microsoft.com/download/symbols</code></li>
</ol>
Close Windbg and open a new session. Ensure that symbols are fixed. If not, examine your  <code>.sympath</code> and ensure it does not differ from the settings that worked in your previous session.
