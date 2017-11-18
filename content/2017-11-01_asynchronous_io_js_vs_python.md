Title: JavaScript Promises are equivalent to Python's asyncio
Date: 2017-11-01T09:00:00+04:00
Category: Logiciel

<img alt="Promises/A+ logo - https://alexn.org/assets/img/2017/then.png" src="{filename}/images/equivalence_then_logo.png" style="float: right; max-width:30%; max-height: 100px; height:auto; padding: 0 10px 1em 1em"/>

JavaScript promises appear to be very different from Python's asyncio.
But they're not that different! The code is written differently (the
*syntax* is different), but what is happening under the hood is
actually the same (the *semantics* are equivalent). The main takeaway
from this post should be:

 * JavaScript programmers should learn from Python (and C#) idioms
   because it will allow them to switch between languages more easily.
 * Python programmers should realize that learning to write code with
   asyncio will help them in JavaScript and other languages.

## Motivation

When I first studied promises in JavaScript and asyncio in Python,
they looked quite different to me. Indeed, promises can use a lot of
anonymous functions and handle errors differently from other
JavaScript code. For example:

    :::js
    function getProcessedData(url) {
      downloadData(url)
      .catch(e => downloadFallbackData(url))
      .then(data => processDataInWorker(data))
    }

Here's how we would write this in Python:

    :::python3
    async def getProcessedData(url):
        try:
            v = await downloadData(url)
        except Exception:
            v = await downloadFallbackData(url)
        return await processDataInWorker(v)

Quite different! The Python code here reads like normal, synchronous
code (with an added `async` and a few `await`s). It's also handling
exceptions using using a mechanism that is standard in many languages.
But I'm here to convince you that those two snippets are actually
equivalent. The main reason why they are is that Python's asyncio and
JavaScript made the same choices.

## The goal: not waiting for I/O

Say you want to write a web server from scratch. One of the first
problems you will want to tackle is: how do I accept connections from
*multiple* clients at the same time?

Well, a given request can trigger a lot of input/output (I/O): reading
files, accessing to the databases and actually communicating with the
client via the Internet. During I/O, the computer has nothing to do:
he can simply take advantage of this to serve the other requests in
the meantime.

Web browsers do the same thing when they want to render a web page:
they don't wait for one resource to arrive to fetch next one. A web
scraper will not wait either to get one page to start fetching the
next one. And in a JavaScript web application, you don't want to
prevent scrolling or other actions from a user when making a request
to the server. All those common use cases are said to be I/O bound,
and you can take advantage of this to improve throughput: each request
won't be faster than before, but all of them will finish much sooner.

The idea is simple enough. But how do we do it?

## Threads to the rescue?

To execute I/O bound requests concurrently, the solution is usually
multithreading. All operating systems will suspend a thread that is
waiting for I/O to allow other threads to get work done. This is
actually very efficient! This is how projects like Apache and uWSGI
work, and it's also supported in nginx. And even though
[multithreading as a programming model has bad reputation][1], in
those cases it's usually just fine because there is very little actual
shared state and your web server takes care of it. (Note that I'm
using the word "thread" here but this also applies to processes,
especially on Linux where threads are just processes with less
overhead.)

[1]: https://stackoverflow.com/questions/1191553/why-might-threads-be-considered-evil

## Event loop

Event loops are an alternative to threads with different advantages.
The language does not have to support threads, which was important for
JavaScript which did not support them initially.

The event loop is well, [a
loop](https://github.com/AndreLouisCaron/a-tale-of-event-loops)! It
will give work to the various functions that are running. When a
function is waiting for I/O, the loop knows that it does not need to
run that function until the I/O request is finished. The initial
mechanism to do this is calling back a function when the event is
ready, as is common in graphical interfaces programming. In the early
2000s, when [Ajax](https://en.wikipedia.org/wiki/Ajax_(programming))
was all the rage, this is already what was happening: when a request
was ready, a callback would be called via `onreadystatechange`.

This event loop paradigm uses only one process and one thread, which
means that there's no context switch needed at the operating system
level, and that running the next bit of code is slightly less costly
than with threads (but other costs can dwarf this benefit).

However, the most interesting thing about using an event loop is the
way it allows programmers to reason about concurrency more easily.
Only I/O operations can take place at the same time, which means that
when you're into a block of code between two I/O operations, you know
that nothing is going to change under you.

## Level of abstraction

However, using callbacks every time I/O is needed is not convenient
and leads to spaghetti code, also known as [callback
hell](http://callbackhell.com/). Gradually, language designers came up
with better ways do this. Here they are, from most low-level to most
convenient:

 1. low-level state machines as provided by mio in Rust
 1. callbacks as first introduced by Node.js
 1. Promises that are now widely used in the JavaScript world
 1. async/await as introduced by C# and supported by Python and ES2017

Each new level is a higher level of abstraction that makes the
resulting code more readable. Interestingly, the highest level of
abstraction can be made as fast as the lowest one, as proven by [Rust
zero-cost futures](https://aturon.github.io/blog/2016/08/11/futures/).

While promises improved the situation greatly, they still require to
learn a different control flow. async/await, on the other hand, allows
the programmer to reason with the usual tools use in synchronous code.

Python is at the async/await level, JS is mostly at the promises level
even though async/await is supported in Node.js since 2016 and is
[coming to browsers](http://caniuse.com/#feat=async-functions).

### Moving from Promises to async/await in JavaScript

Okay, as a JavaScript programmer, why would you want to make the
switch? Remember the code above? No, don't scroll! Here it is:

    :::js
    function getProcessedData(url) {
      downloadData(url)
      .catch(e => downloadFallbackData(url))
      .then(data => processDataInWorker(data))
    }

You can turn into this:

    async function getProcessedData(url) {
      let v;
      try {
        v = await downloadData(url);
      } catch(e) {
        v = await downloadFallbackData(url);
      }
      return await processDataInWorker(v);
    }

This has three benefits.

 * You can use existing idioms such as try/catch blocks.
 * Programmers coming from other languages can get up to speed
   quickly.
 * It's fully compatible with promises, in the sense you can simply
   await on a promise, not only on an async function.

And it's now identical to the Python equivalent:

    async def getProcessedData(url):
        try:
            v = await downloadData(url)
        except Exception:
            v = await downloadFallbackData(url)
        return await processDataInWorker(v)

Okay, a few less braces. :)

## Conclusion

It's really interesting to see that many languages agree that
async/await is the best way to express asynchronous I/O. Other
languages that support this idiom are C# (who introduced it, even if
not using it with an event loop) and Rust ([who considers it essential
to bring futures to
developers](https://github.com/alexcrichton/futures-await).  Other
languages that support this are Dart, Kotlin and Scala: I expect the
list to continue to grow.

It's not that difficult: learn how to use async/await!

Thanks to [Julien Pradet](https://www.julienpradet.fr/) for the
detailed and insightful review that help to massively improve this
blog post.

<!-- vim: spelllang=en
-->
