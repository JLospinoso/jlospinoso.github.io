---
layout: post
title: Setting up a gmock-backed test project in Visual Studio (VS 2013 and gmock v1.7.0)
date: 2015-01-22 23:39
author: jalospinoso
comments: true
categories: [developing, gmock, software, software engineering, test driven development, visual studio]
---
gmock is a unit testing and mocking framework available from Google (<a href="https://code.google.com/p/googlemock/">https://code.google.com/p/googlemock/</a>). Getting it set up and working correctly with a VS 2013 project takes a little bit of ceremony, however--and the errors one can stumble upon are not always the most helpful.

<strong>Get gmock compiled</strong>
<ol>
	<li>Download the latest version of gmock (here we use v 1.7.0, <a href="https://code.google.com/p/googlemock/downloads/detail?name=gmock-1.7.0.zip" target="_blank">https://code.google.com/p/googlemock/downloads/detail?name=gmock-1.7.0.zip</a>)</li>
	<li>Open up the Visual Studio solution (gmock-1.7.0\msvc\2010\gmock.sln)</li>
	<li>Right click gmock &gt; Properties &gt; C/C++ &gt; Code Generation &gt; Runtime Library &gt; Multi-threaded Debug</li>
	<li>Build the solution.</li>
	<li>Ensure that gmock-1.7.0\msvc\2010\Debug contains both gmock.lib and gmock_main.lib. You will link these into your test project.</li>
</ol>
<strong>Configure a test project</strong>
<ol>
	<li>Open your Visual Studio solution (or create a new one). You should NOT create a new project within gmock-1.7.0\msvc\2010\gmock.sln</li>
	<li>Add New Project &gt; Visual C++ &gt; Win32 &gt; Win32 Console Application. Name the project whatever you'd like, but I suggest XXXXXTest, where XXXXX is the name of the project you are testing.</li>
	<li>Right click on your newly created test project &gt; Properties</li>
	<li>Configuration Properties &gt; VC++ Directories
<ol>
	<li>Include Directories &gt; Add the full path to your gmock include and gtest include {e.g.: <strong>C:\Users\jalospinoso\gmock-1.7.0\gtest\include</strong> and <strong>C:\Users\jalospinoso\gmock-1.7.0\include</strong> }. <em>Note: of course, you will want to add the /include folder for your project under test as well. Add this project as a dependency to your test project.</em></li>
	<li>Library Directories &gt; Add the full path to your gmock/gtest artifacts {e.g. <strong>C:\Users\jalospinoso\gmock-1.7.0\msvc\2010\Debug</strong> }</li>
</ol>
</li>
	<li>Configuration properties &gt; Linker &gt; Input &gt; Additional Dependencies
<ol>
	<li>Add <strong>gmock_main.lib</strong> and <strong>gmock.lib</strong>. Note also that you may want to include the name of the project under test here.</li>
</ol>
</li>
	<li> C/C++ &gt; Code Generation &gt; Runtime Library &gt; Multi-threaded Debug</li>
</ol>
<strong>Add a test</strong>

In your test project, create a new source file called "HelloTest.cpp" with the following contents:
<pre>#include "gmock/gmock.h"

TEST(HelloTest, AssertsCorrectly) {
  int value = 42;
  ASSERT_EQ(42, value);
}</pre>
Compile your solution. In your output folder, you will see a new artifact with a name corresponding to your new test project. This executable is a console application that runs all of the tests in your source:
<pre>&gt; MyTest.exe
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
</pre>
&nbsp;
