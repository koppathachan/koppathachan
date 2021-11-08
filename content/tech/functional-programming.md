+++
title = "Functional Programming"
author = ["Akhil Sasidharan"]
date = 2020-09-18T00:00:00+05:30
lastmod = 2021-11-08T23:34:20+05:30
tags = ["currying", "functionalprogramming", "lambdacalculus", "javascript", "js"]
categories = ["programming", "js"]
draft = false
+++

I’ve been going through some functional programming concepts because
I’m on a quest to write better code, and someone told me functional
programming is the way to go. I was intrigued and so, I read about it
and found out that it was based on lambda calculus and was even more
intrigued because it said “calculus”.

The mathematical definition of a function is 'a relationship between two
sets of values such that every element in the first set has a unique
value in the second set'.

Examples:

-   f(x) = x^2 + 1
-   f(x) = cos(x)
-   f(x) = mother of x
-   f(x) = xx
-   f(x) ={ |x| | x ∈ N }, where N is the set of natural numbers.

The values of x makes the first set and the evaluated values makes the
second set.

These functions are called **pure functions** in computer science. A
pure function is a function where the return value is determined by
only its input values without any observable **side effects**. If you
look at the second example above cos(x), will always return the same
value for a given x. For example.

<a id="code-snippet--EgPF"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const add = (a, b) => a+b;
  add(3, 4) // = 7
{{< /highlight >}}

Same function with a side effect.

<a id="code-snippet--EgPFSideEffect"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const add = (a, b) => {
    console.log(`adding ${a} and ${b}.`); // this is a side effect.
    return a+b;
  }
  add(3, 4) // = 7
{{< /highlight >}}

Pure functions are very useful in a special kind of optimisation
called **memoization**. Since for a given value x the value of f(x) will
always remain the same it can be saved for future evaluations and can
speed up expensive function evaluations.

In maths a variable x once assigned a value does not change. That is
to say that when a variable x = 1, it cannot be reassigned to x = 3 or
we cannot mutate x to be ++x. Such an expression would be absurd to a
mathematician. We follow the same rule in functional programming and
it is called **immutability**.

<a id="code-snippet--EgPFImmutable"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const options = { param1: 100 };
  const addParam2 = opts => {
    opts.param2 = 200;
    return opts;
  }
  console.log(addParam2(options)) // { param1: 100, param2: 200 }
  console.log(options) // { param1: 100, param2: 200 }
{{< /highlight >}}

In the above snippet addParam2 function changes the options variable so any
code that uses the options variable after will produce incosistent
values. Using variables this way will also force us to keep track of
all the changes to variables, which is a terrible debugging nightmare.

Instead do this:

<a id="code-snippet--EgPFImmutable"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const options = { param1: 100 };
  const addParam2 = opts => ({ param2: 200, ...opts });
  console.log(addParam2(options)) // { param1: 100, param2: 200 }
  console.log(options) // { param1: 100 }
{{< /highlight >}}

Immutability is important because in an expression such as the following;

<a id="code-snippet--EgPFImmutable2"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const sq = x => x ** 2;
  const equation = x => 3 * sq(x) + 5 * x + 6 // ax^2 + bx + c
  equation(5); // = 106
{{< /highlight >}}

If we replace equation(5) with its value 106 it does not change the
behaviour of the program in anyway. Such expressions are said to be
**referentially transparent**.

In functional programming paradigm functions are first-class which
means they are treated like any other variable. They can be passed to
a function or returned from a function. Such functions are called
**Higher-Order functions**.

<a id="code-snippet--EgPFHigherOrder"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const makeAdjectifier = adjective => noun => `${adjective} ${noun}`;
  const coolifier = makeAdjectifier('cool');
  console.log(coolifier('cat')); //cool cat
{{< /highlight >}}

Consider this program to find the sum of a list of numbers.

<a id="code-snippet--EgPFHigherOrder"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const arr = [ 100, 20, 40, 60, 10, 70 ];
  var sum = 0;
  for (let i = 0; i < arr.length; i++) {
    sum += arr[i]; // we violate the immutability rule here.
  }
{{< /highlight >}}

As you can see we are adding to the variable sum at every iteration
and mutating the value. Consider this solution using the inbuilt
reduce function.

<a id="code-snippet--EgPFHigherOrder"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const arr = [ 100, 20, 40, 60, 10, 70 ];
  const sum = arr.reduce((sum, cur) => sum + curr, 0); //higher-order
						       //reduce function
{{< /highlight >}}

Higher-order functions are essential to writing correct functional code.

I hope this helps in writing better code.
