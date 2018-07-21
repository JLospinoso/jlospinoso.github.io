---
layout: post
title: Visual Studio 2013 and gmock v1.7.0
date: 2015-01-22 23:39
image: /images/gmock.svg
tag: gmock is an excellent unit testing framework, but it takes some effort to get setup
categories: [developing, gmock, software, software engineering, test driven development, visual studio]
---
[1]: https://code.google.com/p/googlemock/
[2]: https://code.google.com/p/googlemock/downloads/detail?name=gmock-1.7.0.zip

gmock is a unit testing and mocking framework available from [Google][1]. Getting it set up and working correctly with a VS 2013 project takes a little bit of ceremony, however--and the errors one can stumble upon are not always the most helpful.

Get gmock compiled
==
Download the latest version of gmock (here we use [v1.7.0][2])

* Open up the Visual Studio solution (`gmock-1.7.0\msvc\2010\gmock.sln`)
* Right click `gmock > Properties > C/C++ > Code Generation > Runtime Library > Multi-threaded Debug`
* Build the solution
* Ensure that `gmock-1.7.0\msvc\2010\Debug` contains both `gmock.lib` and `gmock_main.lib`. You will link these into your test project.

Configure a test project
==

* Open your Visual Studio solution (or create a new one). You should *not* create a new project within `gmock-1.7.0\msvc\2010\gmock.sln`
* `Add New Project > Visual C++ > Win32 > Win32 Console Application`. Name the project whatever you like, but I suggest `XXXXXTest`, where XXXXX is the name of the project you are testing.
* Right click on your newly created test project `> Properties > Configuration Properties > VC++ Directories`
* `Include Directories > ` Add the full path to your gmock include and gtest include (e.g. `C:\Users\jalospinoso\gmock-1.7.0\gtest\include` and `C:\Users\jalospinoso\gmock-1.7.0\include`). *Note: of course, you will want to add the /include folder for your project under test as well. Add this project as a dependency to your test project.*
* `Library Directories > Add the full path to your gmock/gtest artifacts (e.g. `C:\Users\jalospinoso\gmock-1.7.0\msvc\2010\Debug`)
* `Configuration properties > Linker > Input > Additional Dependencies`
* Add `gmock_main.lib` and `gmock.lib`. Note also that you may want to include the name of the project under test here.
* `C/C++ > Code Generation > Runtime Library > Multi-threaded Debug`

Add a test
==

In your test project, create a new source file called `HelloTest.cpp` with the following contents:

```c
#include "gmock/gmock.h"

TEST(HelloTest, AssertsCorrectly) {
	int value = 42;
	ASSERT_EQ(42, value);
}
```

Compile your solution. In your output folder, you will see a new artifact with a name corresponding to your new test project. This executable is a console application that runs all of the tests in your source:

```
> MyTest.exe
Running main() from gmock_main.cc
[==========] Running 1 test from 1 test case.
[----------] Global test environment set-up.
[----------] 1 test from MyTest
[ RUN      ] HelloTest.AssertsCorrectly
[       OK ] HelloTest.AssertsCorrectly(0 ms)
[----------] 1 test from MyTest(0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test case ran. (0 ms total)
[  PASSED  ] 1 test.
```
