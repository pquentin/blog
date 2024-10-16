Title: How do you rate limit calls with aiohttp?
Date: 2017-09-27T09:00:00+04:00
Category: Logiciel

<img title="An ogee-type spillway at the Crystal Dam in Colorado - https://commons.wikimedia.org/wiki/File:Crystaldamogeespillway.jpg" src="{static}/images/ratelimit_dam_spillway.jpg" style="float: right; max-width: 50%; max-height: 300px; height: auto; padding: 0 1em 1em" />

When you first switch away from synchronous sequential code, you
realize that you can send [hundreds of HTTP requests per
second](https://www.artificialworlds.net/blog/2017/06/12/making-100-million-requests-with-python-aiohttp/)
because you don't wait for one request to finish before sending the
next one. These requests can be anything, but I focus on HTTP requests
in this post because they are ubiquitous: you'll use them whether
you're using an API, crawling the web or communicating between
microservices. If the receiving endpoint is prepared, sending many
requests per second might be OK, but in many cases it's not.

In this post, I want to focus on limiting the number of requests per
second. This is related but different from limiting the number of
requests going on at the same time, [which I already
covered](https://quentin.pradet.me/blog/how-do-you-limit-memory-usage-with-asyncio.html)
and which is actually available in aiohttp via
[`limit_per_host`](https://docs.aiohttp.org/en/stable/client.html#limiting-connection-pool-size).
However, the limits set by a given server are based on the number of
requests sent during a specific interval. That's what
[Twitter](https://dev.twitter.com/rest/public/rate-limiting),
[GitHub](https://developer.github.com/v3/search/#rate-limit),
[Salesforce](https://developer.salesforce.com/docs/atlas.en-us.salesforce_app_limits_cheatsheet.meta/salesforce_app_limits_cheatsheet/salesforce_app_limits_platform_api.htm)
and so on do. This post is going to explain how to do this with
aiohttp, which [does not plan to support this
feature](https://github.com/aio-libs/aiohttp/issues/3184), as noted by
mcarans in comments.

## Token bucket

We're going to use the [token bucket
algorithm](https://en.wikipedia.org/wiki/Token_bucket) for rate
limiting. The bucket contains tokens, and you consume one token to
perform one call. If the bucket is empty, you cannot perform more
calls: you need to wait for at least one new token. Before sending
requests, the bucket starts with a number of tokens, and you add a new
token at fixed intervals unless the bucket is full. If you add one
token every 100ms, in the long run you will not make more than ten
requests per second, even if you may have short bursts where you send
more than this.

You may want to read this again before continuing: it's nothing
complicated, but a single read might not be enough!

## Annotating the code

Okay, let's do this. The idea is general, but let's see how to limit
calls done from the aiohttp library. This is going to be a class named
`RateLimiter` that will intercept HTTP GET requests. This class will
[decorate](https://en.wikipedia.org/wiki/Decorator_pattern) the
aiohttp ClientSession class and will be used like this:


    :::python3
    async with aiohttp.ClientSession(loop=loop) as client:
        client = RateLimiter(client)
        async with await client.get(...) as resp:
            ...

So that's a lot like usual aiohttp code, except that we're using
`await client.get()` instead of `client.get()` because it simplifies
our code below.

Let's see the code and comment it as we go, as if we were using
literate programming.

    :::python3
    class RateLimiter:
        RATE = 10
        MAX_TOKENS = 10

We allow 10 requests per second and start with 10 available requests.
For a web crawler, say, we don't want to allow `MAX_TOKENS` to be too
high because you either have a lot of requests to send or nothing to
do, so this would only result in a high burst at the beginning for no
good reason. For other applications, it may make sense to use other
values.

    :::python3
        def __init__(self, client):
            self.client = client
            self.tokens = self.MAX_TOKENS
            self.updated_at = time.monotonic()

As we've seen in our example code above, we receive the client object
and will use it to perform calls. Next, some bookkeeping: we want to
store the current number of tokens (the bucket starts full) and how
much time has elapsed between two buckets fills. We use
`time.monotonic()` to ensure the time always goes forward, which is a
nice property you don't get with `time.time()`.

    :::python3
        async def get(self, *args, **kwargs):
            await self.wait_for_token()
            return self.client.get(*args, **kwargs)

When we call `client.get()`, we first wait for a token to be free,
then go on to perform the actual call. This is remarkably similar to
synchronous code, except that we won't block the whole process during
the `wait_for_token` call: other code may run during the waits. I only
specify `get()` here, but in the real world you would probably want to
cover other methods such as `post()`.

How do we wait for new tokens?

    :::python3
        async def wait_for_token(self):
            while self.tokens <= 1:
                self.add_new_tokens()
                await asyncio.sleep(1)
            self.tokens -= 1

If we have one token ready, we use it and return to the `get()` call.
Note that if we were using threads, "using a token" would need a lock
and would be error-prone. Here, it's easy, you only need to decrement
it because you know that no other code will be executing at the same
time: all other coroutines will be either waiting their turn or
waiting for I/O.

Now, if there are not enough tokens, we need to see if new ones became
available since the last update. If not, just sleep. I'm using a
one-second sleep here, but in real code you would start with a smaller
delay and use exponential back-off.

The only part of the puzzle left is adding new tokens.

    :::python3
        def add_new_tokens(self):
            now = time.monotonic()
            time_since_update = now - self.updated_at
            new_tokens = time_since_update * self.RATE
            if self.tokens + new_tokens >= 1:
                self.tokens = min(self.tokens + new_tokens, self.MAX_TOKENS)
                self.updated_at = now

We simply look at the number of tokens we should add since the last
update and make sure that we do not add more than `MAX_TOKENS`.

The only trick here is that we only change `self.tokens` if it would
allow for a new request, that is, make `self.tokens` go above 1.
Indeed, adding and multiplicating small numbers might lead to
incorrect results due to low clock resolution or underflows. I am not
certain this is actually a problem here, but being sure it never
becomes one is nice!

If you want to use that code, here's a [full
example](https://gist.github.com/pquentin/5d8f5408cdad73e589d85ba509091741)
so you don't have to copy/paste my code above!

## Benefits of asynchronous programming

We don't have to worry about `tokens` or `updated_at` being updated by
something else. The code looks a lot like the synchronous code we're
used to, and we can reason about it the same way. But we still get
great throughput! The only downside is that you need to sprinkle your
code with `async` and `await` which [prevents you to use it in a
synchronous
context](https://quentin.pradet.me/blog/what-color-is-your-python-async-library.html).

Any comments?

If you liked this post, you might like:

 * [How do you limit memory usage with asyncio?](https://quentin.pradet.me/blog/how-do-you-limit-memory-usage-with-asyncio.html)
 * [How do async for loops work in Python?](https://quentin.pradet.me/blog/using-asynchronous-for-loops-in-python.html)

Thanks to Kyrean and Miquel for their insightful comments which led to
corrections in this blog post.

<!-- vim: spelllang=en
-->
