Title: You don't need promises in Python: just use async/await!
Date: 2017-11-01T09:00:00+04:00
Category: Logiciel

<img alt="Promises/A+ logo - https://alexn.org/assets/img/2017/then.png" src="{static}/images/equivalence_then_logo.png" style="float: right; max-width:30%; max-height: 100px; height:auto; padding: 0 10px 1em 1em"/>

If you're coming from a JavaScript background, it's tempting to try to
use the promises that you know and love with Python. That's what I
tried to initially too, and I was surprised to see that promises were
very rarely used in Python. It turns out that promises are not
*pythonic*: I should have used async/await instead. This post explains
why async/await is a better idiom that you can use both in Python and
JavaScript.

I found this surprising at first: I thought that promises were very
different from Python's async/await. But they're not! The code is
written differently (the *syntax* is different), but what is happening
under the hood is actually the same (the *semantics* are equivalent).

The main takeaway from this post should be that it's a **good idea to
use the Python idioms everywhere** because they are easier to learn
and are available in many languages, allowing programmers to switch
between languages more easily. In short, I think you should use
async/await both in JavaScript and Python.

## Motivation

When I first studied promises in JavaScript and async/await in Python,
they looked quite different to me. Indeed, promises can use a lot of
anonymous functions and they handle errors differently from other
JavaScript code. For example:

    :::js
    function getProcessedData(url) {
      downloadData(url)
      .catch(e => downloadFallbackData(url))
      .then(data => processDataInWorker(data))
    }

Here's how we would write this in Python with async/await:

    :::python3
    async def getProcessedData(url):
        try:
            v = await downloadData(url)
        except IOError:
            v = await downloadFallbackData(url)
        await processDataInWorker(v)

Quite different! The Python code here reads like normal, synchronous
code (with an added `async` and a few `await` keywords). It's also
handling exceptions using a mechanism that is standard in many
languages. My goal is to convince you that **those two snippets are
actually equivalent**.

The main reason why they are equivalent is that Python and JavaScript
both use an event loop when doing asynchronous I/O. Let's explain what
that means.

## The goal: not blocking for I/O

First things first, we need to understand what problem such an event
loop is trying to solve.

Say you want to write a web server from scratch. One of the first
problems you will want to tackle is: how do I accept connections from
*multiple* clients at the same time?

The key insight to solve this issue is that a given request can
trigger a lot of input/output (I/O): reading files, accessing
databases and actually communicating with the client via the Internet.
During I/O, the computer has nothing to do: he can simply take
advantage of this to serve the other requests in the meantime.

This idea works in many situations:

 * Web browsers don't wait for one resource to arrive to fetch the
   next one.
 * A web scraper will not wait to fully receive one page to start
   fetching the next one.
 * And in a JavaScript web application, different events can take a
   different amount of time to handle: you don't want to block
   scrolling events even when loading resources from the server.

All those common use cases are said to be I/O bound, and you can take
advantage of this to improve throughput: each request won't be faster,
but all of them will finish much sooner.

The idea is simple enough. But how do we do it?

## Threads to the rescue?

To execute I/O bound requests concurrently, the solution is usually
multithreading. Multithreading actually solves other problems than
performing I/O in parallel, because it also allows to run any code
(not only I/O operations) in parallel on multiple cores.

Anyway, what's important to us here is that all operating systems will
suspend a thread that is waiting for I/O to allow other threads to get
work done. This is actually very efficient! This is how projects like
Apache and uWSGI work, and it's also supported in nginx. And even
though [multithreading as a programming model has bad reputation][1],
in those cases it's usually just fine because there is very little
actual shared state and your web server takes care of it. (Note that
I'm using the word "thread" here but this also applies to processes,
especially on Linux where threads are just processes with less
overhead.)

[1]: https://stackoverflow.com/questions/1191553/why-might-threads-be-considered-evil

## Event loop

However, in JavaScript, threads were initially not supported!
Fortunately, there are other ways to perform multiple I/O operations
in parallel. One way is using an single-threaded event loop.

An event loop is well, a loop! I don't want to go into too much
details, but if you're interested, look at this [toy reimplementation
of an event loop in Python][2] which explains very nicely how
we can write an event loop step by step. However, for our purposes you
only need to know this:

 * The job of the event loop is to wait for I/O without blocking
   the rest of the code.
 * The job of the programmer is to tell the loop when it is going
   perform I/O and what to do when the I/O is over.

[2]: https://github.com/AndreLouisCaron/a-tale-of-event-loops

What's interesting with event loops is the way the [allow programmers
to reason about concurrency more
easily](https://glyph.twistedmatrix.com/2014/02/unyielding.html). Only
I/O operations can be executed in parallel, which means that when
you're into a block of code between two I/O operations, you know that
nothing is going to change under you, which is makes things much
easier than with threads!

Okay, but what does it look like in practice? There are different ways
to program using an event loop, let's look at them.

## Levels of abstraction

The initial mechanism is asking the event loop to call back a function
when an event is ready, as is common in graphical interfaces
programming. In the early 2000s, when [Ajax][3] was all the rage, this
is what was happening: when a request was ready, a callback would be
called via `onreadystatechange`.

[3]: https://en.wikipedia.org/wiki/Ajax_(programming)

However, using callbacks every time I/O is needed is not convenient
and leads to spaghetti code, also known as [callback
hell](http://callbackhell.com/). Gradually, language designers came up
with better ways do this. Here they are, from most low-level to most
convenient:

<!-- Do I have enough courage to add an example for each level? -->

 1. low-level state machines (for example with [mio in Rust][4])
 1. callbacks as in the first versions of Node.js or Ajax
 1. Promises that are now widely used in the JavaScript world
 1. async/await as introduced by C# and supported by Python and ES2017

[4]: http://carllerche.github.io/mio/mio/index.html

Each new level is a higher level of abstraction that makes the
resulting code more readable. Interestingly, the highest level of
abstraction can be made as fast as the lowest one, as proven by Rust
[zero-cost futures](https://aturon.github.io/blog/2016/08/11/futures/)
and [async!/await!
macros](https://github.com/alexcrichton/futures-await).

While promises improved the situation greatly, they still require to
learn a different control flow. async/await, on the other hand, allows
the programmer to reason with the usual tools used in synchronous
code.

Python is at the async/await level, JS is mostly at the promises level
even though async/await is supported in Node.js since 2016 and is
[pretty well supported in browsers][5] and compilable away for IE 11.

[5]: http://caniuse.com/#feat=async-functions

<!--

### What about performance?

Event loops are often sold as a way to get high performance

Threads context switch cost

Extreme case event loops are better

In the general case and with dynamic languages like Python and JS I
don't believe it should be the main reason

Suffice to say that you can get similar performance in both situations
for I/O bound code.
-->

### Moving from Promises to async/await in JavaScript

Okay, as a JavaScript programmer, why would you want to make the
switch from promises to async/await? Remember the code above? No,
don't scroll! Here it is:

    :::js
    function getProcessedData(url) {
      downloadData(url)
      .catch(e => downloadFallbackData(url))
      .then(data => processDataInWorker(data))
    }

You can turn it into this:

    :::js
    async function getProcessedData(url) {
      let v;
      try {
        v = await downloadData(url);
      } catch(e) {
        v = await downloadFallbackData(url);
      }
      await processDataInWorker(v);
    }

This has three benefits.

 * You can use existing idioms such as try/catch blocks.
 * Programmers coming from other languages can get up to speed
   quickly.
 * It's fully compatible with promises, in the sense you can simply
   await on a promise, not only on an async function.

And it's now identical to the Python equivalent:

    :::python3
    async def getProcessedData(url):
        try:
            v = await downloadData(url)
        except Exception:
            v = await downloadFallbackData(url)
        await processDataInWorker(v)

Okay, a few less braces. :)

## Conclusion

It's really interesting to see that many languages agree that
async/await is the best way to express asynchronous I/O in combination
with using an event loop. Other languages that support this idiom are
C# (who introduced it) and Rust ([who considers it essential to bring
futures to developers][6]). Other languages that support this are
Dart, Kotlin and Scala: I expect the list to continue to grow.

[6]: https://github.com/alexcrichton/futures-await

Using async/await is not that difficult: learn how to do it!

Thanks to [Julien Pradet](https://www.julienpradet.fr/) for the
detailed and insightful review that helped to massively improve this
blog post.

<!-- vim: spelllang=en
-->
