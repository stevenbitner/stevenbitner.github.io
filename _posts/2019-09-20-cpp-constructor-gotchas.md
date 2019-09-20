---
title: 'Common Constructor Mistakes in C++'
date: 2019-09-20
permalink: /posts/2019/09/cpp-constructor-gotchas/
tags:
  - C++
  - constructors
  - local vs class variables
---

## Scope
![Don't Hassle Me, I'm Local shirt from What About Bob](images/im-local.jpg)

A very common issue comes from forgetting about the concept of variable scope in C++.
Any time we call a function (and a constructor is just a function, after all) we can declare local variables inside that function.
For example:
```cpp
int Foo::MyCoolFunction (double importantData)
{
	std::string nameOfCoolStuff = "me, probably";
	instanceData = importantData; // instance data, should use this->
	importantData += 1.37;
}
```

The code block above does a few things.
* It declares a locally-scoped variable named `importantData` (this happens in the function declaration).
* It declares another locally-scoped variable named `nameOfCoolStuff`.
* It assigns the value passed in to the function to an instance variable called `instanceData`.
After that, we make some change to `importantData`.
Then the function ends.
That final change did nothing since the variable is local and goes out of scope at the closing brace.

**Note that setting instance data should be done by prefixing with `this->` for clarity.**
It isn't necessary (from the compiler's perspective) to prepend `this->` as long as no local variable has the same name.
This confusion is so easy to avoid with just a few keystrokes.
Get in the habit.

The same applies when the instance variable you wish to set is an object.
The first line in this function declares a locally-scoped variable which gets deleted at the conclusion of the function...typically not what is desired in a constructor.

```cpp
Myclass ()
{
    MyOtherClass reallyImportantStuff(7); // local scope
    this->instanceObject = MyOtherClass(12); // available outside constructor
}
```

## Initialization List
If one class contains another class, you might run into issues if no default constructor exists for the _contained_ class instance.
This is because of how C++ calls constructors.
When a constructor is called, in initializes all instance variables **before** starting the execution of the constructor body.
If a class contains another class, it will initialize the variable by using a default constructor.
This is not an issue if a default constructor exists, but if one does not your code will not compile.

> bar.cpp|9 col 3 error| constructor for 'Bar' must explicitly initialize the member 'myFoo' which does not have a default constructor\
bar.cpp|6 col 7 error| note: member is declared here\
foo.cpp|1 col 7 error| note: 'Foo' declared here


_**foo.cpp**_
```cpp {.line-numbers}
class Foo
{
	private:
		int value;
	public:
		Foo (int value) { this->value = value; }
};

```

_**bar.cpp**_
```cpp {.line-numbers}
#include "foo.cpp"

class Bar
{
	private:
		Foo myFoo;

	public:
		Bar ()
		{
			myFoo = Foo(12);
		}
};

So, what if you cannot make changes to the class to add a default constructor?
Or perhaps you just don't think a default constructor makes sense.

Well, if you know the data that you plan to pass to the inner constructor ahead of time, you can use an initialization in your function declaration.
They are quite simple, for example the constructor in _**bar.cpp**_ above could have been written like this instead:

```cpp
Bar () : myFoo(12) { };
```

This initializes `myFoo` _before_ the constructor body is executed.
Note that in this trivial example, we no longer need to do anything in the constructor body, thus the function is now empty `{ }`.

More than one initialization can be used, just separate them by a comma, e.g.
```cpp
Bar () : myFoo(12), myInt(4), myString("hello world") { };
```

## Questions?
I hope this helps clear some things up.
If you have questions, ask Google or StackOverflow.
If you are my student, come on by my office hours and we'll get it figured out together.