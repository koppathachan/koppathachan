+++
title = "Why use native Promise over Bluebird and other implementations"
author = ["Akhil Sasidharan"]
date = 2020-10-07T03:58:43+05:30
lastmod = 2021-11-08T22:37:43+05:30
tags = ["js", "javascript", "utilfunctions", "programmingbasics"]
categories = ["programming", "js"]
draft = false
+++

> 'Brevity is the soul of wit'

We owe shakespeare the credit for this enduring idiom. The context of
this quote is often lost in its contemporary usage. I am going to
ignore it too, and use it to simply mean 'intelligence or humour is best
expressed briefly'. I am also going to extend this quality, 'soul of wit',
to programming.

The best thing about free and open source software is that there is a
lot of good, clean useful software to choose from to do anything.
Consider the JavaScript ecosystem. Trivial computations like calling a
function multiple times or getting a value from the nested object, or
delaying a function can be done by using generic implementations in
'lodash' package. Even implementations of native methods like forEach,
map, filter, find are found (I guess lodash was pre es3) in
lodash. Coming back to brevity and the soul of wit, I could not bring
myself to include another library for such trivial things. I prefer
custom specific implementations to generic solution especially for
such trivial things. For instance, consider the lodash method
isEmpty(value). lodash.isEmpty(value) looks for all falsey values. Why
must it.

<a id="code-snippet--Eg1"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const isEmpty = arr => arr.length === 0 || Object.keys(arr).length === 0;
{{< /highlight >}}

In most cases its input will be an object or an array. That's readable
and brief. Why must a function that checks if an object is empty,
recieve say, a number. **It is a violation of the Liskov Substitution
Principle aka LSP**.

Another common use case is for Promise. Promises were introduced in
es6 (ES2015). Before that other libraries like Bluebird were
used. Bluebird even performed better than native Promises (before node
10). However, Bluebird also has other functions like Promise.map,
which can be easily implemented using array.map and
async/await. Promise.map also has options like concurrency. But I
cannot see the rationale behind importing a library to use a couple of
its features as opposed spending 15 minutes to implement them
yourself. Here is where that soul of wit is and this habit has made me
a better programmer. A generic solution is not always the best
approach and in those instances I prefer a custom solution.

Nonetheless, I was interested in a generic solution for Promise.map
using modern javscript. Here's what I came up with.

<a id="code-snippet--Eg2"></a>
{{< highlight js "linenos=table, linenostart=1" >}}

  // partition the data into groups of length given by concurrency.
  const partition = (data, concurrency) => Array.from({
    length: Math.ceil(data.length / concurrency)
  }, (_, i) => data.slice(i * concurrency, (i + 1) * concurrency));

  // custom mapReduce takes a mapper function
  // that should be partially applied to return the reduce method.
  const mapReduce = (mapper) => async (
    result, data
  ) => [...await result, ...await Promise.all(data.map(mapper))];

  const map = (
    concurrency = Infinity
  ) => async (
    data, mapper
  ) => partition(data, concurrency).reduce(mapReduce(mapper), []);
{{< /highlight >}}

The above code is simple, follows functional programming paradigm
using modern javascript.

Another irritating example is that of the lodash get method. It allows
us to do this:

<a id="code-snippet--Eg3"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const object = { 'a': [{ 'b': { 'c': 3 } }] };
  get(object, 'a[0].b.c'); // 3
  // and you can pass a default value if the path is not found.
  get(object, 'a.b.c', 'default'); // 'default'
{{< /highlight >}}

In the off chance that I have to use this function, I prefer this.

<a id="code-snippet--Eg3"></a>
{{< highlight js "linenos=table, linenostart=1" >}}
  const makeGet = (def) => (
    obj, ...ks
  ) => ks.slice(1).reduce((
    o, k, _, kss
  ) => (typeof o === 'object' && o[k]) || (kss.splice(1) && def), obj[ks[0]]);

  const get = makeGet('default');

  get(object, 'a', 0, 'b', 'c'); // 3

  get(object, 'a', 0, 'b', 'a'); // 'default'
{{< /highlight >}}

It doesn't take much effort to parse 'a[0].b.c' into the arguments of
this function.

It is always more rewarding to take some time out and implement such
trivial functions. I've learnt a few things from that and it has
defintely made me a better programmer.
