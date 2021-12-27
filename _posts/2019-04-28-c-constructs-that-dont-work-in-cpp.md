---
layout: post
title: C Constructs That Don't Work in C++
image: /images/cconstructs.svg
date: 2019-04-28 06:00
tag: A Survey of the C Constructs That Don't Work in C++
categories: [C, C++, programming, developing, software]
---
[1]: https://www.amazon.com/Programming-Language-hardcover-4th/dp/0321958322/ref=sr_1_1?keywords=stroustrup+C%2B%2B&qid=1556464518&s=gateway&sr=8-1

C++ began life as a fork of C before C was even standardized, so C++ compilers can often directly compile C programs. The price of admission in this case is zero—a programmer can gradually incorporate C++ features into existing C programs as she sees fit. However, C++ isn’t a superset of C, so as a C programmer, it’s worth knowing where disjoints lie. While not exhaustive, this section contains a potpourri of some common problem areas. For a more rigorous treatment, see [The C++ Programming Language by Bjarne Stroustrup (4th Ed.)][1], Chapter 44.

This post discusses three specific C concepts that diverge from C++:

* weak pointer typing
* `enum` values
* function prototypes without arguments

Of course, it's best to write idiomatic C++. But for C programmers who want to adopt C++ in measured steps, it's nice to know what old habits C++ simply won't abide!

# Pointer Typing
C++ regards pointers with stronger typing than C, meaning you can’t implicitly convert from `void*`. For example, a common idiom when using malloc is to lean on implicit conversion:

```c
int *value = malloc(sizeof(int) * 100);
```

This code is dynamically allocating enough space for 100 `int` objects. `malloc` returns a `void` pointer, which is implicitly cast to an `int` pointer and assigned to value.

In C++, this isn’t valid. You could provide an explicit cast, but you must be careful:

```cpp
int *value = (int*)malloc(sizeof(int) * 100);
```

The memory pointed-to by the return value of `malloc` is uninitialized. You should only do two things with such memory: (a) deallocate it, or (b) use it as storage for initializing new objects, e.g. with an in-place `new`.


# Constraint Violations
In older C code, you might find (technically incorrect) constructions like the following:

```c
const int x = 100;
int* x_ptr = &x;
```

This is a constraint violation because you've implicitly shucked off the `const` protections on `x`. Some compilers will let this pass with a warning. GCC 8.3 will, for example, compile this code as C. As C++, it will instead produce an error.

In both C and C++, the remedy is to make the cast explicit. In C++, you can either use an explicit C-style cast or use a `const_cast`.

```cpp
int* x_ptr_1 = (int*)&x; // (1)
int* x_ptr_2 = const_cast<int*>(&x); // (2)
const int* x_ptr_3 = &x; // (3)
```

First, you have the chainsaw approach of using an explicit C-style cast `(1)`. As stated earlier, this is dangerous because it is throwing away the constness of x without being explicitly clear. The `const_cast` is the next approach `(2)`. While this doesn’t get away from the fundamentally dangerous practice of removing `const` from variables, it at least clearly documents a dangerous reinterpretation is being used. If you observe some funny behavior in our program down the road, we have some prime suspects we can scrutinize. Finally, we see the preferred approach, which is taking a pointer to a `const int` `(3)`. This is the safest option, since we are preserving the `const` nature of `x`.

Taking this approach may require some refactoring to deal with a `const` rather than non-`const` pointer, but such a refactor is almost assuredly improving the quality of your code anyway. Remember if you’re in the market for a `const_cast`, you’d better have a very good reason for it, for example interacting with 3rd party or legacy code that you are not allowed to modify. In such a situation where e.g. you know that a function takes a read-only parameter that is not marked const, you can use const_cast to document the point at which you are taking off the safety guards. Such points should be the first place to examine if our program exhibits strange behavior.

C++ also has many additional keywords that C doesn’t, which you can’t use as identifiers in your C++ code. For example, `char *new` wouldn’t be a valid declaration since `new` is a reserved keyword in C++ for dynamic object initialization. The fix is straightforward: simply use a different identifier name.

# enum Values

`enum` values are also a bit different in C++ since they can be any of the C++ integer types, whereas in C an enum always has to be backed by an `int`. In addition to this subtle difference, C++ has stronger typing requirements when handling `enum` values. In C, this code is valid:

```c
enum FooEnum {
  A=0, B, C
};

void enum_cast() {
  int x = A; // (1)
  enum FooEnum foo = 2; // (2)
}
```

Implicit conversions are possible to and from int values in C. You can convert from a `FooEnum` to an `int` `(1)`. You can also convert from an `int` to a `FooEnum` (2). In C++, the first assignment `(1)` is supported while the second `(2)` is not. You can fix this by employing either a C-style cast or a `static_cast`:

```cpp
enum FooEnum foo = static_cast<FooEnum>(2);
```

In this case, you’ve used a `static_cast`, however, this is still a sub-optimal approach from a C++ perspective. The compiler wants to help you enforce your type system! It would probably be better to assign from `FooEnum::C` directly (rather than use its `int` equivalent, 2).

# Function Prototypes without Arguments

Function prototypes that don’t contain arguments are handled differently in C and C++. In C, this will compile:

```c
void fn() { }

int main() {
  fn(42);
}
```

This isn’t necessarily a good thing since it’s unclear if the caller really meant to invoke `fn` with a parameter. It’s more likely to be an error that the C compiler didn’t help us with. Fortunately, this unintuitive “feature” of C wasn’t inherited by C++, so this snippet won’t compile in C++. In C++, you must specify your function with the correct parameter types. To do this for the above example, you could modify `fn` to take an `int`:

```cpp
void fn(int);
```

Now the compiler can enforce that you're invoking our functions with the intended parameters. C's handling of parameterless functions is a historic hold-over that has the potential to cause serious headaches.

# C99

If you're a C programmer wanting to venture into the C++ world, you'll have to give up these habits. It's probably fair to say that, after getting used to the C++ way of doing business, you won't miss them too much!

In contrast, some of the features introduced in the C99 standard made a number of welcome improvements to C. One such improvement is the designated initializer, which provides some syntactic sugar to the programmer when initializing members:

```
struct Address {
  char street[256];
  char city[256];
  char state[256];
  int zip;
};

struct Address white_house = {
  .street = "1600 Pennsylvania Avenue NW",
  .city = "Washington",
  .state = "District of Columbia",
  .zip = 20500
};
```

These are not available in C++; however, the constructor has similar functionality with much stronger guarantees called class invariants. A class invariant is some behavior about the class that doesn’t change throughout its lifetime. The programmer has absolute autonomy in deciding what these invariants are, so they are an extremely powerful feature of C++.

> Note: Designated initializers will likely be available in C++20.

The C99 standard also introduced the `restrict` keyword. If pointer `x` is marked `restrict`, the programmer promises that no other pointer will refer to the object pointed to by `x`, which can potentially enable the compiler to emit more efficient code. This keyword doesn’t exist in the C++ standard, although some compilers may support it (for example msvc supports `__restrict`).

Finally, you turn to flexible array members. Since C99, it is valid to include a dimensionless array as the last member of a `struct`:

```
struct Bar {
  int x;
  char y[];
};

void make_bar() {
  struct Bar* bar = malloc(sizeof(int) + 256 * sizeof(char));
  bar->y[255] = 42;
  free(bar);
}
```

C++ doesn’t have support for such members, but there are higher-level C++ constructs that can provide similar functionality with greater safety--such as STL containers and `std::unique_ptr`.

# Conclusion

While C++ largely supports well-written C programs, there are some corner cases to watch out for. Hopefully this post will help those veteran C programmers looking to dabble in the brave new world of C++ a mapping of those territories.
