---
title: 'Uninitialized variables in C++'
date: 2019-09-20
permalink: /posts/2019/12/cpp-uninitialized-variables/
tags:
  - C++
  - undefined behavior
---

C++ won't always work the way you expect, and that is by design.
It's a remarkably rigid language and it requires that a programmer cross all of their _f_'s and dot all of their _j_'s.

## Undefined Behavior
![Schrodinger's cat joke](/images/schrodingers-cat.jpg)

For the authoritative reference, see https://en.cppreference.com/w/c/language/behavior.

### When Might This Happen?
If you declare variables without initializing them, you'll inevitably have problems.
Some of those problems will immediately cause segmentation faults and lead you down your happy debugging path.
Others can be quiet and might deceive you into thinking that your program works as expected.

Note that not all OSs and not all compilers do the same thing.
The examples below might not give strange results on Mac OS, but then will on Cent OS, or vice-versa.

## How Do I Detect These Problems?
The following program will run without error but the behavior is undefined and will differ from system to system.

```cpp
#include <iostream>

int main ()
{
	bool b;
	if (b)
	{
		std::cout << "True\n";
	}
	else
	{
		std::cout << "False\n";
	}

	return 0;
}
```

Compiling your code with the `-Wuninitialized` (or `-Wall` which will report other warnings as well) flag will detect the use of uninitialized scalar values. 
> **foo.cpp:** In function **`int main()`**:\
> **foo.cpp:6:2:** _warning:_ `b` is used uninitialized in this function [-Wuninitialized]\
>   if (b)\
>   ^\
> False

### This Won't Work On the Hard Stuff
Ask yourself what the following snippet will output.

```cpp
bool* uninitializedBool = new bool[10]; // uninitialized local variable

if (uninitializedBool[0])
{
	std::cout << "true\n";
}

if (!uninitializedBool[0])
{
	std::cout << "false\n";
}
```

If you guessed
> true\
> false

you are pretty good.
Whether you guessed that or not, you are likely pretty confused.
How can a variable be both `true` and `false`?
This is captured by the statement from the cpp resource above as _unspecified behavior_.
Specifically,
> Each unspecified behavior results in one of a set of valid results and may produce a different result when repeated in the same program.

Not precisely a Schrodinger's Cat analog, but close enough to include a funny comic in the intro.

## I'd Never Do That
Clearly the example above is trivial for the sake of showing a minimal example.
In reality, this might be forgetting to initialize the final element in an array, or the final dimension in a multi-dimensional array.
Having a `for` loop end early, or forgetting to increment an index will land you in this situation.

## So, How _Do_ I Detect These Problems?
When in doubt, `valgrind`.
Running Valgrind on the program produced above would give something along the lines of

> ==2766== Memcheck, a memory error detector\
==2766== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.\
==2766== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info\
==2766== Command: ./a.out\
==2766==\
==2766== **Conditional jump or move depends on uninitialised value(s)**\
==2766==    at 0x40074C: main (in /home/sbitner/a.out)\
==2766==\
==2766== **Conditional jump or move depends on uninitialised value(s)**\
==2766==    at 0x400769: main (in /home/sbitner/a.out)\
==2766==\
false\
==2766==\
==2766== HEAP SUMMARY:\
==2766==     in use at exit: 10 bytes in 1 blocks\
==2766==   total heap usage: 1 allocs, 0 frees, 10 bytes allocated\
==2766==\
==2766== LEAK SUMMARY:\
==2766==    definitely lost: 10 bytes in 1 blocks\
==2766==    indirectly lost: 0 bytes in 0 blocks\
==2766==      possibly lost: 0 bytes in 0 blocks\
==2766==    still reachable: 0 bytes in 0 blocks\
==2766==         suppressed: 0 bytes in 0 blocks\
==2766== Rerun with --leak-check=full to see details of leaked memory\
==2766==\
==2766== For counts of detected and suppressed errors, rerun with: -v\
==2766== Use --track-origins=yes to see where uninitialized values come from\
==2766== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)\

## Questions?
I hope this helps clear some things up.
If you have questions, ask Google or StackOverflow.
If you are my student, come on by my office hours and we'll get it figured out together.
