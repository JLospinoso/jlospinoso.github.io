---
layout: post
title: Common x86 Calling Conventions
image: /images/grumpy_calling_convention.jpg
date: 2015-04-04 16:00
tag: Understanding cdecl, stdcall, and fastcall is critical to understanding x86 assembly
categories: [assembly, c, developing, software]
---
[1]: http://en.wikipedia.org/wiki/X86_calling_conventions
[2]: http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html
[3]: http://www.nasm.us/
[4]: https://jlospinoso.github.io/developing/software/software%20engineering/reverse%20engineering/assembly/2015/03/06/reversing-with-ida.html
[5]: https://msdn.microsoft.com/en-us/library/zkwh89ks.aspx
[6]: https://github.com/JLospinoso/x86CallingConventions
[7]: https://msdn.microsoft.com/en-us/library/a90k134d.aspx
[8]: http://www.nasm.us/doc/nasmdoc7.html
[9]: http://stackoverflow.com/questions/2035568/why-do-stacks-typically-grow-downwards
[10]: https://msdn.microsoft.com/en-us/library/c1h23y6c.aspx

In some situations, we may want to integrate some hand-crafted object code into our project. This is most productively accomplished by writing [Intel x86 assembly][2] and assembling it into object code via an assembler like [NASM][3]. In other situations, we may be looking at disassembly (that may have come from C, Assembly, or both). Further, on x86 architectures all of our C/C++ code gets compiled into [Intel x86 object code][2]. So if we are [reverse engineering unmanaged code][4], it will also be critical to understand how functions call each other at the lowest level. In either case, we must know exactly how functions are called, otherwise it would be difficult to make heads or tails of a program's flow. In this post, I'll go over the three most common calling conventions. You'll need to know only the very basics of assembly to follow along.

# Function calls on x86 architectures
The x86 architecture doesn't have any concrete notion of a function call in the same sense that high level software languages do--we work with a series of jumps back and forth between blocks of code. Consider the following example:
	
	add eax, ebx
	jmp MultiplyEaxByTwo
	Continue:
	; ...
	
	MultiplyEaxByTwo:
	shl eax,2
	jmp Continue

After adding the `ebx` register to `eax`, we multiply `eax` by two, then continue executing `;...`. This is a simple analogy to a function call in assembly.

The preferred way to perform function calls is a little different.  We typically use `call` and `ret`:

	add eax, ebx
	call MultiplyEaxByTwo
	; ...
	
	MultiplyEaxByTwo:
	shl eax,2
	ret

`call` differs from `jmp` in one important way--it pushes the address of the next instruction to be executed after returning (here `;...`) onto the stack. Why? Because `ret` pops this value off the stack and sets our instruction pointer `eip`. This is why we didn't have to label `;...` as `Continue` in the above example. `ret` does all of the work of returning execution to the callee for us.

In the above examples, we actually created our own calling convention. The function `MultiplyEaxByTwo` takes in one parameter, `eax`, which it modifies in place. When calling assembly-to-assembly, we are free to come up with whatever calling conventions we think are convenient. It is once we want to interact with higher level languages like C that we must pay careful attention to protocols! If we wanted to call `MultiplyEaxByTwo` from C, how could we do it? We'd want to make a call like:

	extern void MultiplyEaxByTwo(int *ptr);
	// ...
	unsigned int x = 10;
	MultiplyEaxByTwo(&x);
	printf("This value should be twenty: %u\n", x);

C is going to set up the stack in a specific way before `call MultiplyEaxByTwo`, it is going to expect the stack to look a specific way after `MultiplyEaxByTwo` executes a `ret`, and (for functions with a return value) it is going to look in a very specific place for the result.

These specifics define the *calling convention*.
	
# The Big Three
There are [lots and lots][1] of calling conventions. Fortunately, there are only three that predominate in C (there is one other, the `thiscall` convention, which is used extensively in `C++` but is outside of the scope of this tutorial). All of the code from this section is arranged in a VS2013 solution available on [GitHub][6].

Calling conventions answer three important questions:

* What do the stack and registers look like when the function is called?
* What do the stack and registers look like when the function returns?
* Where is the result stored?


## Investigating `cdecl`
First, up is `cdecl`, which is the [default calling convention for C and C++][5]. Consider a function taking three `int`s and returning the sum:

	__declspec(dllexport) int __cdecl MyCdecl(int a, int b, int c);

The `__cdecl` keyword specifies that `MyCdecl` adheres to the `cdecl` calling convention. The `__declspec(dllexport)` keyword allows us to [export the function][7] in a DLL, so that we can test our function:

	TEST_CLASS(MyCdeclTest)
	{
	public:

		TEST_METHOD(MyCdeclAddsCorrectly)
		{
			auto result = MyCdecl(30, 8, 4);
			Assert::AreEqual(42, result);
		}
	};

We implement `MyCdecl` in assembly to illustrate the `cdecl` calling convention:

	GLOBAL _MyCdecl
	EXPORT _MyCdecl
	_MyCdecl:
		xor eax,eax
		add eax,[esp+4]
		add eax,[esp+8]
		add eax,[esp+12]
		ret

Let's break down `_MyCdecl` line by line:

1. All `cdecl` functions start with an underscore. `GLOBAL` tells NASM that `_MyCdecl` is a global symbol.
2. `EXPORT` tells NASM we are writing a DLL, and this function is to be exported. We [still need to mark it GLOBAL][8]!
3. `_MyCdecl:` is a label that tells NASM where our function starts.
4. `xor eax,eax` sets `eax` to zero
5. `add eax,[esp+4]` adds the value on the stack 4 bytes above the stack pointer `esp`
6. Same as 5, but with an 8-byte offset.
7. Same as 5, but with a 12-byte offset.
7. Pop `eip` off the stack, i.e. return to the caller.

According to `cdecl`, the stack will look like this when the top of `_MyCdecl` executes our test method `MyCdeclAddsCorrectly`. Each block `[   ]` represents four bytes.

				 Low memory
	[  RP  ] <-- ESP
	[  30  ]
	[   8  ]
	[   4  ]
	[ ...  ]
				 High Memory

`RP` denotes the address of the *return pointer*, i.e. the value that was pushed onto the stack when `call` was executed. This is the address where our function will return execution to. 

The arguments have been pushed onto the stack from right to left, so that the first argument is at the lowest memory address (recall that the [stack grows towards lower memory addresses][9]). EIP's value is the address of `RP`, so `[esp+4]` corresponds with our first argument, `[esp+8]` with our second, and so on.

OK, so we are zeroing out `eax`, then adding each of the three arguments. But how do we return this value? It turns out that in `cdecl` all we had to do was store the return value into `eax`. That's it!

One subtle feature of `cdecl` to note is that the caller will have to *clean up the stack*, i.e. return `esp` to the correct value, after `ret`. In fact, this is the major difference between `cdecl` and `stdcall`, which we examine next.
	
## Investigating `stdcall`
To investigate `stdcall`, let's create a similar function:

	__declspec(dllexport) int __stdcall MyStdcall(int a, int b, int c);
	
The big difference here is in the `__stdcall` keyword. As you might expect, this tells C that our function implements the `stdcall` convention. Our test will be nearly identical to the previous section's:

	TEST_CLASS(MyStdCallTest)
	{
	public:
		
		TEST_METHOD(MyStdCallAddsCorrectly)
		{
			auto result = MyStdcall(30, 8, 4);
			Assert::AreEqual(42, result);
		}

	};

We mentioned that the major difference between `cdecl` and `stdcall` is that the callee (our function) must clean up the stack. How is this done? Well, it turns out that `ret` can take an argument corresponding with the number of bytes to add to `esp` after popping off the return pointer:

	GLOBAL _MyStdcall@12
	EXPORT _MyStdcall@12
	_MyStdcall@12:
		xor eax,eax
		add eax,[esp+4]
		add eax,[esp+8]
		add eax,[esp+12]
		ret 12

Why add? Examine what we would like the stack to look like after returning:


				 Low memory
	[  RP  ]
	[  30  ]
	[   8  ]
	[   4  ]
	[ ...  ] <-- ESP
				 High Memory

ESP needs to increment a total of 16 bytes--4 for the return pointer, then 12 for the arguments. That's it!

The naming convention for `stdcall` functions is to prepend an underscore and append an `@` followed by the number of bytes that must be cleaned off the stack, i.e. the number of arguments times four.

## Investigating `fastcall`
The final x86 calling convention you're likely to run into when looking at C programs is the `fastcall` convention. Like `stdcall`, the callee must clean the stack. The major difference between the two is that the first two arguments will *not* be present on the stack. Instead, they are shoved into the general purpose registers `ecx` and `edx`. Keeping data in registers rather than in memory can be much faster (hence the name `fastcall`).

Our function declaration appears as follows:

	__declspec(dllexport) int __fastcall MyFastcall(int a, int b, int c);

The test follows from the above two sections:

	TEST_CLASS(MyFastcallTest)
	{
	public:

		TEST_METHOD(MyFastcallAddsCorrectly)
		{
			auto result = MyFastcall(30, 8, 4);
			Assert::AreEqual(42, result);
		}
	};

When `MyFastcall` executes, the stack will look like this:

				 Low memory
	[  RP  ] <-- ESP
	[   4  ]
	[ ...  ]
				 High Memory

`ecx` will contain `30` and `edx` will contain `8`. Our implementation is then

	GLOBAL @MyFastcall@12
	EXPORT @MyFastcall@12
	@MyFastcall@12:
		xor eax,eax
		add eax,ecx
		add eax,edx
		add eax,[esp+4]
		ret 4

Notice that we must only clean 4 bytes from the stack, since two of our arguments are in registers rather than the stack. The naming convention for `fastcall` functions is to prepend an `@`, append an `@`, then append the number of bytes to clean from the stack.

# Wrapping up

Notice that each of our test cases looks exactly the same--from C, the calling convention really doesn't matter. 

![Test Results]({{ site.url }}/images/CallingConventionTests.jpg)

Where it matters is when we dive underneath the surface and get into the x86 object code. Our three conventions have these main features:

* `cdecl`: caller cleans the stack, all arguments are pushed onto the stack from right to left. Names formatted like `_foo`
* `stdcall`: callee cleans the stack, arguments the same as `cdecl`. Names formatted like `_foo@20`.
* `fastcall`: callee cleans the stack, first two arguments in `ecx`, `edx`, rest onto the stack from right to left. Names formatted like `@foo@12`

It is worth getting very familiar with these three calling conventions! If you do *any* assembly programming, reverse engineering, or vulnerability research, it is an absolutely critical skill to have.

## Some gotchas
There is some ceremony you must go through to get NASM assembling as part of your MSBuild process. I recommend starting from [https://github.com/JLospinoso/x86CallingConventions][6], which contains the following bit of magic within the DLL's project file:

	<ItemGroup>
	<CustomBuild Include="CallingConventions.nasm">
	  <FileType>Document</FileType>
	  <Message Condition="'$(Configuration)|$(Platform)'=='Release|Win32'">Compiling %(Identity)</Message>
	  <Message Condition="'$(Configuration)|$(Platform)'=='Debug|Win32'">Compiling %(Identity)</Message>
	  <Command Condition="'$(Configuration)|$(Platform)'=='Debug|Win32'">nasm -f win$(PlatformArchitecture) -Xvc -o $(IntermediateOutputPath)\%(Filename).obj %(Identity)</Command>
	  <Outputs Condition="'$(Configuration)|$(Platform)'=='Debug|Win32'">$(IntermediateOutputPath)\%(Filename).obj;%(Outputs)</Outputs>
	  <Command Condition="'$(Configuration)|$(Platform)'=='Release|Win32'">nasm -f win$(PlatformArchitecture) -Xvc -o $(IntermediateOutputPath)\%(Filename).obj %(Identity)</Command>
	  <Outputs Condition="'$(Configuration)|$(Platform)'=='Release|Win32'">$(IntermediateOutputPath)\%(Filename).obj;%(Outputs)</Outputs>
	</CustomBuild>
	</ItemGroup>
	
As you're building out your DLL, `dumpbin` is an [indispensable tool][10] for determining if you are having export-related issues:

	dumpbin /EXPORTS foo.dll
	
will tell you whether you've successfully exported your function, and with what calling convention (information you can glean from the exported name).

