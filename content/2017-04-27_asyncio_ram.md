Title: How do you limit memory usage with asyncio?
Date: 2017-04-27T09:00:00+04:00
Category: Logiciel

One of the first hurdles that you can encounter when trying out
asyncio is "asyncio eats all my memory!". Indeed, to keep you CPU
busy, you're encouraged to launch a lot of coroutines simultaneously.
Coroutines don't use a lot of memory by themselves, but what you're
doing inside them can use quite a lot of memory. 

<img alt="Rail semaphore (CC BY 2.0, Dave-F at http://flickr.com/photos/92163630@N00/3060839)" src="{filename}/images/asyncio_ram_semaphore.jpg" style="float: left; max-width:100%; height:300px; padding: 0 1em 1em 0"/>

I worked recently on an asynchronous crawler that runs in a Kubernetes
cluster. As with all microservices that run in our cluster, I
configured a memory limit (400Mib here). Even with rate limiting to
play nice with external web sites, scheduling a few thousand
coroutines that need to fetch a web page, parse it, and call two other
microservices for normalization and storage means we're going to hit
the memory limit fast. In the best case, your crawler gets killed, and
can no longer work.

One work around is obviously to give more memory to my crawler. It
would have worked in my case, but in the vast majority of situations,
my crawler only used 200Mib. What I actually wanted to prevent are
memory spikes, because there's no reason to burn a lot of memory for
ten minutes if I'm going to be idle for one hour after that.

Now that we've understood the problem, what is the solution? Well,
it's simply to limit the number of coroutines that are running
simultaneously. How do you do that? That was not obvious to me. It
turns out you to be the classical concurrency solution: a semaphore!

Using a semaphore can seem a bit counter-untuitive. After all, we're
using only a single thread, so trying to prevent multiple things to
run at once can seem weird. But what you want to prevent is to *start*
too many coroutines will others are running, so it does make sense.

Ok, how does it work?
[asyncio.Semaphore](https://docs.python.org/3/library/asyncio-sync.html#asyncio.Semaphore)
provides two methods: an `acquire` coroutine, and a release function.
The idea is to not start work unless you can acquire the semaphore,
and the semaphore limits the number of coroutines that can acquire it
simultaneously. Here's a simple example:

    :::python
    async def do_work(semaphore):
        await semaphore.acquire()
        print('start work')
        await asyncio.sleep(1)
        # optionally do a lot of work that will consume memory
        print('end work')
        semaphore.release()

Since we always want to acquire first and release last, we can use a
context manager:

    :::python
    async def do_work(semaphore):
        async with semaphore:
            print('start work')
            await asyncio.sleep(1)
            # optionally do a lot of work that will consume memory
            print('end work')

It conveys our intent more clearly and prevents bugs. (Indeed,
[decoupling acquire and release is only a recipe for failure
anyways](http://web.stanford.edu/~engler/deviant-sosp-01.pdf).) To add
one more level of safety, we'll use a `BoundedSemaphore` that ensure
that we can never call `release()` more than `await acquire()`.

Let's see a full example now. We have this initial code:

    :::python
    import asyncio

    async def do_work():
        print('start work')
        await asyncio.sleep(1)
        # optionally do a lot of work that will consume memory
        print('end work')

    async def main():
        tasks = []
        for i in range(100):
            tasks.append(asyncio.ensure_future(do_work()))
        await asyncio.gather(tasks)

    if __name__ == '__main__':
        loop = asyncio.get_event_loop()
        loop.set_debug(True)
        loop.run_until_complete(main())

If you want to only run ten `do_work()` at a time, you now simply do
this:

    :::python
    import asyncio

    async def do_work(semaphore):
        # new: only enter if semaphore can be acquired
        async with semaphore:
            print('start work')
            # optionally do a lot of work that will consume memory
            await asyncio.sleep(1)
            print('end work')

    async def main():
        tasks = []
        # new: instantiate a semaphore before calling our coroutines
        semaphore = asyncio.BoundedSemaphore(10)
        for i in range(100):
            # new: pass the semaphore to the coroutine that will limit
            # itself
            tasks.append(asyncio.ensure_future(do_work(semaphore)))
        await asyncio.gather(tasks)

    if __name__ == '__main__':
        loop = asyncio.get_event_loop()
        loop.set_debug(True)
        loop.run_until_complete(main())

And that's it! Even if we still start 100 `do_work()` coroutines at
once, only ten of them will actually work at the same time.

You may have one last question: how do you choose the semaphore
initial counter value? That depends on the memory you are willing to
use, the memory each coroutine uses and the memory that the rest of
your code uses. While you could compute an optimal number, I found
that trial and error works well here.

<!-- vim: spelllang=en
-->
