---
title: 'Debugging Segmentation Faults in C++'
date: 2020-10-01
permalink: /posts/2020/10/cpp-debugging-seg-faults/
tags:
  - C++
  - debugging
  - segmentation fault
  - gdb
  - core dump
---

Segmentation faults in C++ are a sign that you are trying to do hard things.
Congratulations!
Now, let's take a peek at how to start debugging.

![Homer Simpson Y2K joke](https://media.giphy.com/media/xT5LMYqgQk7yAhvvRm/giphy.gif)

## Valgrind
Never underestimate the easiest option.
If trying to run `a.out` results in a seg fault, try running it through Valgrind to see if you can gain some insight.

`./a.out`
> Segmentation fault: 11

`valgrind ./a.out`
> ==57489== Memcheck, a memory error detector  
 ==57489== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.  
 ==57489== Using Valgrind-3.16.0.GIT and LibVEX; rerun with -h for copyright info  
 ==57489== Command: ./a.out  
 ==57489==   
 --57489-- run: /usr/bin/dsymutil "./a.out"  
 warning: no debug symbols in executable (-arch x86_64)  
 0 1 2 3 4 5   
 ==57489== **Invalid read of size 4**  
 ==57489==    at 0x100003C5B: operator<<(std::ostream&, SimpleGrid const&) (in ./a.out)  
 ==57489==    by 0x100003933: main (in ./a.out)  
 ==57489==  Address 0x1200000002 is not stack'd, malloc'd or (recently) free'd  
 ==57489==   
 ==57489==   
 ==57489== Process terminating with default action of signal 11 (SIGSEGV)  
 ==57489==  **Access not within mapped region at address 0x1200000002**  
 ==57489==    at 0x100003C5B: operator<<(std::ostream&, SimpleGrid const&) (in ./a.out)  
 ==57489==    by 0x100003933: main (in ./a.out)  
 ==57489==  If you believe this happened as a result of a stack  
 ==57489==  overflow in your program's main thread (unlikely but  
 ==57489==  possible), you can try to increase the size of the  
 ==57489==  main thread stack using the --main-stacksize= flag.  
 ==57489==  The main thread stack size used in this run was 8388608.  
 ==57489==   
 ==57489== HEAP SUMMARY:  
 ==57489==     in use at exit: 90,922 bytes in 164 blocks  
 ==57489==   total heap usage: 178 allocs, 14 frees, 95,666 bytes allocated  
 ==57489==   
 ==57489== LEAK SUMMARY:  
 ==57489==    definitely lost: 0 bytes in 0 blocks  
 ==57489==    indirectly lost: 0 bytes in 0 blocks  
 ==57489==      possibly lost: 4,392 bytes in 5 blocks  
 ==57489==    still reachable: 86,530 bytes in 159 blocks  
 ==57489==         suppressed: 0 bytes in 0 blocks  
 ==57489== Rerun with --leak-check=full to see details of leaked memory  
 ==57489==   
 ==57489== For lists of detected and suppressed errors, rerun with: -s  
 ==57489== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 1 from 1)  
 Segmentation fault: 11

Bold font added above for instructive purposes.
This Valgrind output indicates that my program is trying to access something outside a mapped region.
This almost certainly comes from defining an array of size **n** and trying to access element **n+1** which would be index **n**.
So, in just a second or two, we know to check our array bounds, most likely due to an improperly defined `for` loop.

## GDB
GDB is a powerful debugger that allows a programmer to step through their code line by line and probe any variable for its value at that step of execution.
It has a lot of capability beyond what can be addressed in a simple primer.
A very useful cheat sheet that I always have a printed copy of on my desk can be found here [https://darkdust.net/files/GDB Cheat Sheet.pdf](https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf).

First and foremost, GDB will need some specific information injected into the executable that needs to be debugged.
This requires compiling all of our code with the `-g` flag.
The best way to do this is by adding it to your `CXXFLAGS` variable in your Makefile.
That ensures that all automatically created \*.o files are also built using the `-g` flag.
You should also delete the old \*.o files before rebuilding.
This is a great time to run `make clean`, assuming you have a well-defined `clean` rule in your Makefile.

Now that you have recompiled with `-g` flags, you can fire up the debugger.
The `-tui` flag below opens the source code in the top half of the screen which is great for adding some context to where the program is in the execution.
`gdb -tui a./out`

If files were built with `-g`, the symbol table should load and GDB is ready to use.
The first thing that is typically needed is to add one or more breakpoints.
These can be added to method names, or to lines of code (if you specify a line that cannot break, the next breakable line is used).

### Breaking on a specific line of source code
To halt execution at line 12 of test.cpp, you would simply type `break test.cpp:12`.
Now when you type `run`, GDB starts executing and will halt at line 12 (or the next breakable line) so that you can inspect the status of your program.

### Breaking on a method
If you suspect a specific method or function in your code, you can halt execution and inspect whenever that method is called.
In order to inspect the method `Search` within the `BinarySearch` class, I would type `break BinarySearch::Search(int*, int, int, int)`.
As you might have guessed, this is a great time for tab-completion which GDB is great at.
Just start typing the class or method name and hit **tab** in order to fill in the rest.
Now when you type `run`, GDB starts executing and will halt the first (and every) time that method is called.

### Stepping through
The commands needed to restart execution after a breakpoint has been reached are:
- `next` - This goes to the next line of code, but will not dive into functions.
- `step` - This goes to the next line of instruction. This might be inside of a function call or elsewhere in the code tree.
- `continue` - This runs the program until the end of execution, or until a breakpoint is reached.

### Looking at values
The simple act of stepping through code often helps figure out the problem.
"This should not go inside that `if` statement"...well, it does, so figure out why.

Sometimes, it is not enough to just know the current location of your code execution.
Luckily, we can print out values of our variables.
`print myVar` would print the value of `myVar`.
`print &myVar` would print the address of `myVar`.
Pointers will print the address by default.
To see their value, you would need to dereference, e.g. `print *myPointer`.
As you might have noticed, the print statement is identical to sticking in `std::cout` statements, except that you don't need to guess in advance, you can _poke around_ until you find the values you need.
Print will also let you call methods, so `print myObject.GetSomeData()` would print the results of the method call.

When breaking on a method call, the arguments passed to the call are printed automatically.
For the BinarySearch breakpoint we mentioned earlier, when the breakpoint is triggered, the output might read:
> Breakpoint 1, BinarySearch::Search (listOfNumbers=0x7fffffffc720, left=7, right=10, searchKey=10) at binary-search.cpp:5

## `std::cout`
"Did this line run?"
You can always try sending output to screen in a bunch of places to approximate the power you can get via step-through debugging with a tool such as [gdb](#gdb).
If you have peppered more than a couple such statements in your code for the purpose of tracking down a single bug, you're likely to be better off just using GDB.
If you have a bunch of output statements in your code as a standard part of your development process, you are almost certainly missing out on the real power of unit tests and TDD in particular.

Depending on where your output statements are, and where the seg fault occurs, it is possible that output is still in the buffer when the crash occurs.
This can be misleading since you won't see the output and might think the error must have occurred first.
To flush the buffer you can add the `std::endl` output sequence to your `cout`s.
This should ensure that the output that gets generated does indeed make it to your screen rather than dying on the way.
So, replace output such as
```cpp
std::cout << "Foo value: " << foo << '\n';
```
with
```cpp
std::cout << "Foo value: " << foo << std::endl;
```

## Commenting Out Code
The "_old reliable_", never forget that a program with zero lines of code doesn't have any seg faults.
Commenting out blocks of code can sometimes be a very useful strategy to narrowing down the location of a bug.
Lines 100-200 are commented out but you still have the seg fault, it must be someplace else.

When using this approach, don't forget about the power of binary search.
Searching for a seg fault in 100 lines of code shouldn't require "toggling" smaller and smaller blocks of code on and off more than a handful of times.


## Comparison

| Tool        | Does not require recompiling | Fast             | Easy to use | Easy to understand |
| :---        | :--:                         | :---:            | :---:       | :---:              |
| Valgrind    | X                            | X                | X           |                    |
| GDB         |                              |                  |             | X                  |
| `std::cout` |                              | If you get lucky | X           | X                  |
| Commenting  |                              | X                | X           | X                  |
