Title: What Color is Your Python async Library?
Date: 2017-04-05T09:00:00+04:00
Category: Logiciel

Since I've began to program more than ten years ago, I've heard
everywhere that synchronous, sequential programming is a way of the
past. In a
[manycore](https://en.wikipedia.org/wiki/Manycore_processor) future,
we will need to write programs that execute in parallel. But aside
from [embarassingly parallel
programs](https://en.wikipedia.org/wiki/Embarrassingly_parallel), this
has always been considered to be too difficult to be worth it.

One of the first issues to deal with is that we often confuse
parallelism, concurrency and multithreading. While [concurrency
enables
parallelism](https://blog.golang.org/concurrency-is-not-parallelism),
[multithreading](https://en.wikipedia.org/wiki/Multithreading_(software))
is only one way to attain concurrency. And [programming with threads
is very hard](https://glyph.twistedmatrix.com/2014/02/unyielding.html)
because of all the things that we must consider when using them.

What alternative models of concurrency do we have? While threads are
general, there are other models that make different trade-offs.
Indeed, most problems that need concurrency are IO-bound, not
CPU-bound. When you're reading a file or making an HTTP request, your
CPU stays idle most of the time. How can we take advantage of this?

The answer is [asynchronous
I/O](https://en.wikipedia.org/wiki/Asynchronous_I/O). During that HTTP
request, your program can continue doing something else. While the
idea is old, it gained popularity recently. node.js, released in 2009,
made this idea very popular. But it also made it easy to enter
[callback hell](http://callbackhell.com/). In 2012, C# 5.0 introduced
async/await to the world. It was a great step forward that was adopted
in Python and JavaScript. async/await, in some sense, is just a way to
let the compiler write the callbacks itself. (Read about
[continuation-passing
style](https://en.wikipedia.org/wiki/Continuation-passing_style) if
you would like to learn more about this.) But it does makes writing
asynchronous I/O much more approachable, and I would not consider
writing asynchronous code without that feature.

Asynchronous I/O has attracted a lot of praise and criticism. The best
criticism came from Bob Nystrom who wrote a very influential post
named [What Color is Your
Function?](http://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/)
in which he explains that asynchronous programming forces us to divide
the world in two. For every synchronous library that does I/O you now
need an asynchronous library too. This happens in the [node.js
standard library](https://nodejs.org/api/fs.html), but also in the
Python world where the community reimplements many libraries using
asyncio, for example as part of the [aio-libs
project](https://github.com/aio-libs).

If the cost is so great, why continue to write asynchronous code?
Speed is not the (only) answer. After all, [carefully written threaded
code can also handle more than 10K concurrent
connections](http://stackoverflow.com/questions/17593699/tcp-ip-solving-the-c10k-with-the-thread-per-client-approach/17771219#17771219),
despite what [some proponents of asynchronous I/O
say](https://lwn.net/Articles/692254/). However, asynchronous
programming makes it [much easier to reason about concurrency for
large programs with lots of
users](https://glyph.twistedmatrix.com/2014/02/unyielding.html) while
still writing fast code. And this is a huge deal.


## The many worlds

However, in Python, the situation is even worse than the theoretical
two worlds. We have way more than two worlds.

First, there's synchronous code. The requests library was written with
only synchronous support in mind, and there's a lot of work needed
before async can happen. Among other things, [Python 2 support has to
be
dropped](https://github.com/kennethreitz/requests/issues/1390#issuecomment-225395648),
and it won't be happen before a few years. It *is* possible to use a
threadpool to simulate synchronous code in a asynchronous way, but
it's only a measure of last resort.

Second, there's *callback-based* asynchronous programming. This style
was made popular with node.js before promises started to gain
traction.  In Python, that's mostly Twisted, but also the
callback-based part of asyncio, the standard library module. The two
are mostly compatible, and in fact a project named
[txaio](https://txaio.readthedocs.io/en/latest/) allows to write code
that's compatibly between the two. Twisted is [not going
anywhere](https://lwn.net/Articles/692254/), mostly because it
includes a lot of [stuff that does not exist elsewhere
yet](https://glyph.twistedmatrix.com/2014/05/the-report-of-our-death.html).
The main issue with the callback approach is that while it's an
improvement over using threads, the code can very easily end up as
spaghetti code that is very hard to follow.

Third, we have the async/await part of asyncio in the standard
library. This syntax allows the code to be way more natural to read
because it's close to sequential code and is much easier to reason
about. It attracted a lot of mindshare because it was the standard
approach, and this is what the ambitious
[aio-libs](https://github.com/aio-libs) project support. That includes
aiohttp, but also aiomysql, aiopg, aioredis, and so on. Unfortunately,
[asyncio is a complex beast with many concepts to
understand](http://lucumr.pocoo.org/2016/10/30/i-dont-understand-asyncio/).
Starting Python 3.6, asyncio is no longer included on a [provisional
basis](https://docs.python.org/3.5/glossary.html#term-provisional-api)
and is now guaranteed to be stable. It will continue to be the first
place newcomers come to. Not going away either.

Fourth, we have [curio](https://github.com/dabeaz/curio). It reuses
the syntax introduced with asyncio. After all, in Python [async/await
is a
protocol](https://mail.python.org/pipermail/async-sig/2016-November/000166.html)
and asyncio is only a library, and nothing prevents building others to
continue to innovate in the space. Unlike asyncio, it does not try to
support callbacks. Surprisingly, this allows curio users to [write
less code which is more
correct](https://vorpus.org/blog/some-thoughts-on-asynchronous-api-design-in-a-post-asyncawait-world/).
Unfortunately, curio is still small and does not have all the
libraries offered by the options above.

And there are others. eventlet, gevent, Tornado,
[trio](https://github.com/python-trio/trio/). The list never ends. And
they're not compatible: writing code that works between all of them is
close to impossible.

## How do we cope?

We could lament at the specific Python situation, but it actually
enabled a lot of innovation. Each new library appears to be better
than the next. But still, how do we cope? One way would be keep the
async functions in our code to a minimum. In [The Function Colour
Myth](https://lukasa.co.uk/2016/07/The_Function_Colour_Myth/#how-to-live-with-coloured-functions),
Cory Benfield argues that with care, no more than 10% of your codebase
needs to be `async` functions. But this is only true in new codebases.

In many existing codebases, we don't try to isolate I/O code. When we
do try, we only isolate I/O in a specific class or function, but it
still gets called all over the place. requests is *the* well-designed
library in Python, but it stills [YOLOs bytes into and out of sockets
like nobody's
business](https://github.com/kennethreitz/requests/issues/1390#issuecomment-224975630)
according to one of its maintainers (Cory Benfield, again).

The best way to isolate I/O is to perform *no I/O at all*. This
radical idea from... Cory Benfield allowed to write
[hyper-h2](https://python-hyper.org/h2/en/stable/) and
[h11](https://github.com/njsmith/h11). They're implementations of
HTTP/1.1 and HTTP/2 that don't perform any I/O. How is this possible?
Well, they introduce a toolkit which can be used in any Python code,
be it synchronous, using async/await, curio, callbacks, and so on. It
allows the Python ecosystem to start sharing code again. And since
there's no I/O in those implementations, they are easy to test, too.
h11, for example, has 100% coverage for both statements and branches.

The idea is generic, and a library named
[python-effect](https://github.com/python-effect/effect) tries to help
with it. The names comes from programming language research, but what
is basically does is isolating I/O and state-manipulation code in
objects. Those objects capture the I/O computation, which allows to
perform I/O in a really small part of the code. Time will tell how
successful this library becomes.

Anyway, if you write Python asynchronous code, I encourage you to
think about staying compatible. Please don't get stuck in a single
world, the other ones have many things to offer!

<!-- vim: spelllang=en
-->
