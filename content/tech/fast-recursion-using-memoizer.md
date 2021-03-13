+++
title = "Memoization, with a js implementation that caches recursive calls"
author = ["Akhil Sasidharan"]
date = 2020-09-20T23:50:30+05:30
lastmod = 2021-03-13T22:57:59+05:30
tags = ["functionalprogramming", "lambdacalculus", "javascript", "js", "memoization", "memoizer", "recursion", "fastrecursion"]
categories = ["programming"]
draft = false
weight = 2004
+++

Memoization is an **optimization technique** used in functional
programming to speed up execution by storing the results of resource
expensive function calls. When the function is called again with the
same input the stored result is fetched and returned. This is possible
in functional programming languages because of the use of **pure
functions** as discussed in this [post](https://akhilsasidharan.in/posts/functional-programming/). Purely functional languages such
as Haskell has inbuilt support for memoization. In javascript, using a
mutable map (object, map, caches) we can implement a memoization.

Memoization is especially useful in recursive functions. Writing code
the functional way makes my code expressive and testable. However, as
I will demonstrate now, in javascript (and most other impure
functional languages) recursion is horrendously slow. Recursion is
essential to functional programming.

Let's look at the **fibonacci series**. The mathematical formula of
which is,

F<sub>n</sub> = F<sub>n-1</sub> + F<sub>n-2</sub>, where F<sub>0</sub> = 0, F<sub>1</sub> = 1

or

F<sub>n</sub> = F<sub>n-1</sub> + F<sub>n-2</sub>, when n > 1, and

F<sub>n</sub> = n, when n <= 1

Looking at this equation one can see why recursive function appeals
here. Look at the analogous js code.

<a id="code-snippet--EgFibRec"></a>
```js
  const fib = (n) => n > 1 ? fib(n - 1) + fib(n - 2) : n;
```

To calculate the fibonacci of 40 the above function took more than a
second. Beyond fibonacci of 50 the output depends on what video I am
playing on my laptop. The non recursive but super fast code looks like
this.

<a id="code-snippet--EgFib"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const uglyFib = (n) => {
      let a = 0, b = 1, c, i;
      if (n == 0) return a;
      for (i = 2; i <= n; i++) {
      c = a + b;  // We violate immutability rule here
      a = b;      // and here
      b = c;      // and here
      }
      return b;
  };
{{< /highlight >}}

It doesn't look anything like its mathematical representation. I
wished it had the charming good looks of its recursive counterpart to
go with its dashing performance. That wish was granted; we
have memoization.

But first let me show you what happens while calculating the fibonacci
value of 5. fib(5).

<a id="code-snippet--Fib5Tree"></a>
```nil
fib(5)
|
+--fib(4)
|  |
|  +--fib(3)-------------------------------1
|  |  |                                    |
|  |  +--fib(2)-----------------1          |
|  |  |  |                      |          |
|  |  |  *--fib(1)              |          |
|  |  |  |                      |          |
|  |  |  *--fib(0)              |          |
|  |  |                         |          |
|  |  +--fib(1)                 |          |
|  |                            |          |
|  +--fib(2)--------------------2          |
|     |                         |          |
|     *--fib(1)                 |          |
|     |                         |          |
|     *--fib(0)                 |          |
+--fib(3)-----------------------|----------2
|  |                            |
|  +--fib(2)--------------------3
|  |  |
|  |  *--fib(1)
|  |  |
|  |  *--fib(1)
|  |
|  +--fib(1)
```

From the above tree we can see that fib(3) is called 2 times, fib(2)
is called 3 times, fib(1) 6 times and fib(0) 2 times. Memoization is
how we avoid these repeated calls by saving the result the first time
fib(n) is called. When the result is returned its value is cached in
an object with the key as the functions input (n in this case). This
can be reused by subsequent calls to the function with the same input.

Let's look at a basic implementation of memoization.

<a id="code-snippet--EgMemoizedFib"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const fibonacci = (n, memo = {}) {   // provide a default object as
				       // cache.
    if (memo[n]) { return memo[n]; }   // looking in the cache.
    if (n <= 1) { return 1; }
    memo[n] = fibonacci(n - 1, memo) + // save the result and pass the
	      fibonacci(n - 2, memo);  // cache object.
    return memo[n];                    // return the result
  }
{{< /highlight >}}

The above implementation not only caches the result of fibonacci(5),
but also intermediate results of fibonacci(4), fibonacci(3) and all
the rest of them.

Some npm modules like fast-memoize and memoize provide generic
implementations to memoize any function like this.

<a id="code-snippet--EgMemoizedFib"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const memoize = require('fast-memoize')
  const fn = function (one, two, three) { /* ... */ }
  const memoized = memoize(fn)
  memoized('foo', 3, 'bar')
  memoized('foo', 3, 'bar') // Cache hit
{{< /highlight >}}

But they do not cache intermediate results like we saw above in the
custom implementation. While I do like custom implementations over a
generic solution, I attempted a generic solution that caches
intermediate values. That is, if I call fib(5) the memoized value of
fib(5) will cache fib(4), fib(3), fib(2), fib(1), fib(0) before the
function fib(5) has returned, which is speeds up some recursive
functions.

<a id="code-snippet--EgMemoizedFib"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  export const memoize = (func, cache = Object.create(null)) => {

  // here we do some magic to sanitize body and arguments recieved from
  // the func.toString() call. Then we return a new function as shown
  // below.

    return new Function('cache',
  `
  return function ${func.name} (${args}) {
    let result = cache[JSON.stringify([${args}])]
    if (result) { return result; }
    result = ${body}
    cache[JSON.stringify([${args}])] = result;
    return result;
  }
  `)(cache);
  };
{{< /highlight >}}

Not an elegant solution admittedly, but it does the job, given the
limitations of javascript. You can explore the full code at
<https://github.com/sasidakh/memoizer>.

I tested this implementation where the fibonacci of 40 was calculated
and it was only 4 times slower (the first time it was called) than its
non recursive counter part as opposed to being nearly 40000 times
slower.

| Without recursion            | : | : | x               |
|------------------------------|---|---|-----------------|
| With recursion               | : | : | ~ 39000x slower |
| Memoized recursion           | : | : | ~4x slower      |
| Memoized recursion ran twice | : | : | ~42x faster     |

**Wo-hoo! My code is faster thanks to memoization**

You can run the tests on the [repo](https://github.com/sasidakh/memoizer) to understand it better.
