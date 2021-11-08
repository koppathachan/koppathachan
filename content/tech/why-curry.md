+++
title = "Why currying?"
author = ["Akhil Sasidharan"]
date = 2020-09-16T21:07:46+05:30
lastmod = 2021-11-08T23:34:18+05:30
tags = ["currying", "functionalprogramming", "lambdacalculus", "javascript", "js"]
categories = ["programming", "js"]
draft = false
+++

Currying is the transformation of a function written like this

<a id="code-snippet--EgCurry"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const add = (a, b) => a+b;
  add(3, 4) // = 7
{{< /highlight >}}

Into this

<a id="code-snippet--EgCurry"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const add = a => b => a+b;
  add(3)(4) // = 7
{{< /highlight >}}

Which allows me to do this.

<a id="code-snippet--EgCurry"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const add = a => b => a+b;
  const add3 = add(3); // this can be read as 3 + b;
  add3(1) // = 4
{{< /highlight >}}

Here I partially applied 3 to the function add to get add3 a specific function that adds 3 to its input.

You can compose functions like these.

<a id="code-snippet--EgCurry2"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const longerThan = n => word => word.length > n;
  const words = ['something', 'schadenfreude', 'ambivalent',
      'good', 'bad', 'preposterous'];
  const find = words => lengthCond => words.filter(lengthCond);
  find(words)(longerThan(7));
{{< /highlight >}}

Now that just reads 'find words longer than seven'.

<a id="code-snippet--EgCurry2"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const greet = message => name => `${message} ${name}!`;
  const sayHelloTo = greet('Hello');
  sayHelloTo('Akhil'); // Hello Akhil!
  const sayWelcomeTo = greet('Welcome');
  sayWelcomeTo('Mamta'); // Welcome Mamta!
  const sayGoodDayTo = greet('Good Day');
  sayGoodDayTo('Ritika'); // Good Day Ritika!
{{< /highlight >}}

It's easy on the eyes isn't it.

I especially like this:

<a id="code-snippet--EgCurry3"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const double = x => x * 2;
  const triple = x => x * 3;
  // This is a bit tricky to read admittedly, but look what it produces
  const pipe = (...fns) => n => fns.reduce((total, f) => f(total), n);
  const doubler = pipe(double); // same as double
  const quadrupler = pipe(double, double);
  const sextuple = pipe(double, triple);
  quadrupler(3); // 12
  sextuple(5); // 30
{{< /highlight >}}

This is called composition.

Here's another fun one.

<a id="code-snippet--EgCurry4"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const subString = start => len => str => str.substr(start, len);
  const lowerCase = str => str.toLowerCase();
  const firstCharacter = str => substring(0)(1)(str);
  const firstCharacterAsLower = str => lowerCase(firstCharacter(str));
{{< /highlight >}}

In conclusion, currying if used properly makes code really readable and can produce powerful abstractions through compositions and partial application of functions. I will explore partial application examples in my next post.
