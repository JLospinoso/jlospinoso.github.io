---
layout: post
title: Aligning bytes in VS2013
date: 2015-02-16 00:27
author: jalospinoso
comments: true
categories: [c++, developing, software, visual studio]
---
In some situations where fine-grained control over memory is required, we must be careful about how we pack bytes into memory. These alignment issues can arise e.g. with particular kinds of processors or when crafting low-level input/output for hardware.

C++11 offers the <strong>alignof</strong>/<strong>alignas</strong> operators (<a title="SC22/WG21/N2341" href="http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2341.pdf" target="_blank">http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2341.pdf</a>), but unfortunately they are not implemented in the Visual Studio 2013 compiler (<a href="https://msdn.microsoft.com/en-us/library/hh567368.aspx" target="_blank">https://msdn.microsoft.com/en-us/library/hh567368.aspx</a>). It is possible to lean on <strong>std::align</strong>, however (<a href="http://en.cppreference.com/w/cpp/memory/align" target="_blank">http://en.cppreference.com/w/cpp/memory/align</a>).

Suppose we have a 32 byte region of memory, and we need to always align our entries along 8-byte boundaries. In the example, we use up the first 9 bytes of the memory:

<code>#define MEM_SIZE 32
#define HEADER "\x01\x02\x03\x04\x05\x06\x07\x08\x09"
#define HEADER_LENGTH 9</code>

<code> char data[MEM_SIZE] = { 0 };
memcpy(data, HEADER, HEADER_LENGTH);</code>

Our <strong>data</strong> buffer looks like this:

<a href="https://jalospinoso.files.wordpress.com/2015/02/f11.jpg"><img class="alignnone size-medium wp-image-121" src="https://jalospinoso.files.wordpress.com/2015/02/f11.jpg?w=300" alt="f1" width="300" height="67" /></a>

The next region of data needs to be aligned at an 8 byte boundary (i.e. at the 16th byte), and we can lean on <strong>std::align</strong>:

<code>#define ALIGNMENT 8
#define BUFFER "\x10\x11\x12\x13\x14\x15\x16\x17"
#define BUFFER_LENGTH 8</code>

<code> void *endOfHeader = &amp;(data[0]) + HEADER_LENGTH;
std::size_t space = MEM_SIZE - HEADER_LENGTH;
void *nextAvailableByte = std::align(ALIGNMENT, BUFFER_LENGTH, endOfHeader, space);
memcpy(nextAvailableByte, BUFFER, BUFFER_LENGTH);</code>

<strong>std::align</strong> takes four arguments:
<ol>
	<li>The size of the alignment (in our case, 8 bytes)</li>
	<li>The length of the data you'd like to write (in our case, also 8 bytes).</li>
	<li>The (possibly mis-aligned) pointer to the next valid address to write buffer to.</li>
	<li>How much space is left in <strong>data</strong>. If  <strong>std::align</strong> cannot find a spot to insert into <strong>data</strong>, it will return a null pointer (i.e. there was not enough space left, as <strong>BUFFER_LENGTH</strong> was too large).</li>
</ol>
The result of our <strong>memcpy</strong> is <strong>BUFFER</strong> copied nicely along our 8 byte boundary:

<a href="https://jalospinoso.files.wordpress.com/2015/02/f2.jpg"><img class="alignnone size-medium wp-image-122" src="https://jalospinoso.files.wordpress.com/2015/02/f2.jpg?w=300" alt="f2" width="300" height="67" /></a>

A complete example is available at <a href="http://pastebin.com/gb9E5QdW" target="_blank">http://pastebin.com/gb9E5QdW</a>
