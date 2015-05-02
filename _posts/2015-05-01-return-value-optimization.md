---
layout: post
title: Return Value Optimization
---

### Outline ###

* motivation: learning a language
  - more than syntax, underlying behavior
  - compiler, architectures, conventions

* return value optimization
  - what it is
  - why it's helpful
  - when it's important to use it

* show code
  - how it works

* empirical data and graphs
  - possibly strace for malloc

* conclusion


After about a year and a half of coding in C++, things are finally starting to click. Learning the syntax is just the tip of the iceburg, what's more important is the underlying behavior and possible side effects of our choices. For example, what happens if you don't make an accessor of a member function const? Usually, nothing - but as time goes on, and more engineers modify the same component, the chance of introducing a side effect, or using the accessor incorrectly goes up. Making the member function const is a signal to the compiler to tell you better how you screwed up. And it'll continue to do that for the future contributors to the project. The latent benefits and optimizations of the compiler are worth exploring and understanding. <em>One</em> such optimization is called Return Value Optimization.

As [wikipedia](wikipedia.org/return-value-optimization) states, <abbr title="Return value optimization">RVO</abbr> is when an extra parameter is passed into the function to be populated with the return value instead of being populated in the return value register. This parameter is a pointer to the variable being assigned the output of the function. This can be used to save an extra copy via assignment, as well as simplifying the code and consequently, more readable.

Let's say you want to write a range function in C++ which populates a vector with a range of integers. A simple range function might have a min and max argument, with the assertion that max must be greater than min. Since we are returning a container that can dynamically grow and shrink, thus makes use of heap memory and executes a deep copy within a copy constructor or assignment operator, we'd want to avoid copies <bold>as much as possible</bold>. The standard way to do this is pass in the vector that we want to populate by reference, and return void.

{% highlight c++ %}

// Assertion max > min
static void range(std::vector<int>& v, int min, int max);


// Usage

int main()
{
    std::vector<int> v;
    range(v,10,20);
    std::cout<<v.size();

    // Output should be 10
}

{% endhighlight %}


The 'wrong' way to do this would be return it by value (which in theory instantiates a deep copy), and have the callee assign the return value to another vector, either by assignment or copy constructor, thereby instantiating a second deep copy. 


{% highlight c++ %}

// Assertion max > min
static  std::vector<int> range(int min, int max);


// Usage

int main()
{
    std::vector<int> v = range(10,20);
    std::cout<<v.size();

    // Output should be 10
}

{% endhighlight %}

With return value optimization, which is supported by any modern C++ compiler, the two range functions are <em>equivalent</em>. In the underlying assembly code, no copy operations of the vector are occuring, making it both efficient and in my opinion, easier to read.


But how do we prove to ourselves that this is actually doing what I say it does? There are a couple of ways: 1) to peruse the generated assembly of both functions and deduce their equivalency, 2) to run it through strace and view how many system calls to malloc are being made, or 3) benchmark it to procure empirical results and plot it.

I originally tried reading through the generated assembly to no avail. The majority of the code was standard library code bloat, making it a nuisance to parse through. Then I tried running both implementations through a variety of range sizes, and comparing it to a range implementation that on purpose executes a deep copy on the return value.

{% highlight c++ %}

#include <vector>
#include <iostream>
#include <assert.h>

std::vector<int> rvo-range(int min, int max)
{
    std::vector<int> result;
    assert(max > min);
    while (min < max)
    {
	result.push_back(min++);
    }

    return result;
}

void standard-range(std::vector<int>& result,int min, int max)
{

    assert(max > min);
    while (min < max)
    {
	result.push_back(min++);
    }
}

void without-rvo-range(std::vector<int>& result,int min, int max)
{

    std::vector<int> temp;
    assert(max > min);
    while (min < max)
    {
	temp.push_back(min++);
    }
    result = temp;
}
{% endhighlight %}

-----

Next week we'll be talking about Unordered Maps vs Vectors
<strong>TO DO: Add links </strong>
