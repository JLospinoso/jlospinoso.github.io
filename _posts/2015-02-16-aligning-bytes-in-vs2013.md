---
layout: post
title: Aligning bytes in VS2013
date: 2015-02-16 00:27
image: /images/aligning_bytes.svg
tag: Byte alignment can be much simpler with C++11
categories: [c++, developing, software, visual studio]
---
[1]: https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2341.pdf
[2]: https://msdn.microsoft.com/en-us/library/hh567368.aspx
[3]: https://en.cppreference.com/w/cpp/memory/align
[3]: https://gist.github.com/JLospinoso/1abf58847c41b908764568a477256f46
[db1]: {{site.url}}/images/2015-02-16_1.jpg "Data Buffer 1"
[db2]: {{site.url}}/images/2015-02-16_2.jpg "Data Buffer 2"

In some situations where fine-grained control over memory is required, we must be careful about how we pack bytes into memory. These alignment issues can arise e.g. with particular kinds of processors or when crafting low-level input/output for hardware.

C++11 offers the `alignof`/`alignas` operators ([OpenSTD][1]) but unfortunately they are not implemented in the Visual Studio 2013 compiler ([VS2013][2]). It is possible to lean on `std::align`, however ([CPP Align][3]).

Suppose we have a 32 byte region of memory, and we need to always align our entries along 8-byte boundaries. In the example, we use up the first 9 bytes of the memory:

```cpp
#define MEM_SIZE 32
#define HEADER "\x01\x02\x03\x04\x05\x06\x07\x08\x09"
#define HEADER_LENGTH 9

char data[MEM_SIZE] = { 0 };
memcpy(data, HEADER, HEADER_LENGTH);
```

Our **data** buffer looks like this:

![db1]

The next region of data needs to be aligned at an 8 byte boundary (i.e. at the 16th byte), and we can lean on `std::align`:

```cpp
#define ALIGNMENT 8
#define BUFFER "\x10\x11\x12\x13\x14\x15\x16\x17"
#define BUFFER_LENGTH 8

void *endOfHeader = &amp;(data[0]) + HEADER_LENGTH;
std::size_t space = MEM_SIZE - HEADER_LENGTH;
void *nextAvailableByte = std::align(ALIGNMENT, BUFFER_LENGTH, endOfHeader, space);
memcpy(nextAvailableByte, BUFFER, BUFFER_LENGTH);
```

`std::align` takes four arguments:
1. The size of the alignment (in our case, 8 bytes)
2. length of the data you'd like to write (in our case, also 8 bytes)
3. The (possibly mis-aligned) pointer to the next valid address to write buffer to
4. How much space is left in `data`. If `std::align` cannot find a spot to insert into `data`, it will return a null pointer (i.e. there was not enough space left, as `BUFFER_LENGTH` was too large).
The result of our `memcpy` is `BUFFER` copied nicely along our 8 byte boundary:

![db2]

A complete example is available on [github][3].
