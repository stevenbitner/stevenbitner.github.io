---
title: 'Operator Overloading in C++'
date: 2019-09-13
permalink: /posts/2019/09/cpp-operator-overloading/
tags:
  - C++
  - operators
  - friend keyword
---

What follows is an example of an overloaded operator for a class named `Foo`.

## Motivation
Operators such as `+` are self explanatory, how would we add two Stocks together in a portfolio without it.
Why bother overloading the output operator, can't we just use `ToString()`?

### Syntactic sugar
Which block of code below is easier to read:
```cpp
std::cout << students[i].ToString()
          << " is in " << classes[i].ToString()
          << " taught by " << instructors[i].ToString();
```

```cpp
std::cout << students[i]
          << " is in " << classes[i]
          << " taught by " << instructors[i];
```
It's a small thing, and not much of a change (syntactic sugar rarely is) but code will be written once and read many times.
Making the reading of the code easier on the eyes can make a large difference in time.

### Data types
A `ToString()` method will require us to cast some class or instance data into strings.
This might not be a big deal, calling `to_string` on an integer is easy enough, but in the context of templates we can have some problems.
Try calling `to_string` on a string and watch your program crash.
Using streams is how we can make our implementation more flexible, and that's never a bad thing.

### It's standard
Did we call the method `ToString`, or `GetString`, or `toString` or `PrintObject`...you get the point.
Each class that gets built might have a different variation on what is created.
Sure, you can find the API in the header file and know the name, but why bother having to look it up each time you want to display an object.
You can also try to fix this through coding standards, but enforcement can become burdensome and a standard for outputting things already exists in C++, it is `<<`.

## Sample from a header (.hpp) file
```cpp
friend std::ostream& operator<< (std::ostream& out, const Foo& objectToDisplay);
```

## Sample from an implementation (.cpp) file
```cpp
std::ostream& operator<< (std::ostream& out, const Foo& objectToDisplay)
{
	out << "This Foo is great. It's value is " << objectToDisplay.value;
	return out; // must return the stream object to enable chaining
}
```

## LHS and RHS
A function such as `myFoo.toString()` has a clear context and function name since it is all **explicit**.
In the case of an operator, in math or in C++, there is no context, simply a left-hand side and right-hand side operand.
Thus, the `operator<<` function name is _implied_ by the call.
The arguments to the function are the left-hand side and right-hand side, respectively.
In the case of the output stream operator (`<<`), the left-hand side is the desired output stream, while the right-hand side is the object to be output.

## Chaining
The return type of `std::ostream&` is no accident.
Essentially, we make the call with the stream on the left and the object on the right.
Those two statements are then combined with the return becoming the left hand side of the next call.

In math, we add integers in the following way: `1+2+3+4 = 3+3+4 = 6+4 = 10`.
At each step, we add the two leftmost integers and the result is an integer which is now the leftmost call to the next operation.
The same principle applies to any overloaded operator in C++.

## The "Friend Zone"
We're computer scientists and software developers, we've all been in the friend zone, but what does that mean in C++?
Notice in the sample implementation for `operator<<` [above](#sample-from-an-implementation-cpp-file) we access the data member `value` of the `Foo` instance.
If we've encapsulated our data properly, then those data members are private, thus not accessible outside of our `Foo` class.
But this operator is one that we _trust_ so we're going to allow it to access those members anyway.
You can argue the merits of this approach versus its violation of pure encapsulation, but that is another discussion that is primarily a battle of louder opinions.

Why do we need it though, I mean this function would (presumably) be in the implementation file for `Foo`.
Because of how `<<` is called, it isn't called within the **context** of a class instance.
Thus, when we define it, it is not resolved to the scope of the class.
Above, see that the function declaration isn't given like this:
```cpp
friend std::ostream& Foo::operator<< (std::ostream& out, const Foo& objectToDisplay)
```
It's given without the `Foo::` scope resolution, like this:
```cpp
friend std::ostream& operator<< (std::ostream& out, const Foo& objectToDisplay)
```
That's because we don't say:
```cpp
Foo myFoo;
myFoo.<<;
```
but rather

```cpp
Foo myFoo;
std::cout << myFoo;
```

## Questions?
I hope this helps clear some things up.
If you have questions, ask Google or [Stackoverflow](https://stackoverflow.com/questions/4421706/what-are-the-basic-rules-and-idioms-for-operator-overloading).
If you are my student, come on by my office hours and we'll get it figured out together.