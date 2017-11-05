---
layout: post
title:  "Python 3's Killer Feature: asyncio"
date:   2017-10-23
author:     "Kyle Ding"
header-img: "img/Event-Loop.png"
catalog: true
tags:
    - Python
---

A [massive](https://news.ycombinator.com/item?id=12829759) [debate](https://news.ycombinator.com/item?id=13722852) in the python community about python2/3 has been raging for years. The main reason for python3 to diverge was to provide unambiguous types to handle unicode, strings and bytes ([more here](https://news.ycombinator.com/item?id=10752028)), but recently there’s been a bigger divergence that’s gone largely unnoticed. python3.4 introduced [the asyncio module](https://docs.python.org/3/library/asyncio.html) and python3.5 gave it a new syntax that is built into the language. Asyncio has done for python what node did for javascript. Asynchronous programming in python enables you to do new and powerful things, but like all technologies there are tradeoffs.


## **Performance**

We use uvloop to speed up our asyncio code ([link](https://magic.io/blog/uvloop-blazing-fast-python-networking/)):

> *“uvloop makes asyncio fast. In fact, it is at least 2x faster than nodejs, gevent, as well as any other Python asynchronous framework. The performance of uvloop-based asyncio is close to that of Go programs.”*

Asyncio can significantly improve throughput on the same hardware. You don’t have to use semaphores, you get access to shared memory, and it’s relatively easy to code. However this won’t improve latency since await-ed functions still have to wait for IO. While our await-ed functions wait for IO, control is automatically returned to the event loop so other code can run.

For applications with lots of IO, the savings can be substantial. Imagine a service like pingdom that visits a ton of websites to be sure they’re still up. Serially making HTTP requests and waiting for responses would be painfully slow. Before asyncio, you had to use [python’s threading module](https://docs.python.org/3/library/threading.html), which has several limitations

 

## **Example Code**

Let’s start, by showing what some asynchronous python 3.5+ code looks like:


**Regular Snippet **

```python
async def get_popular_words(words_to_ignore, histogram_size):

    # make asynchronous call
    data = await slow_function_that_waits_for_io()

    # calculate histogram for data
    histogram = {}
    for word in data.split():
        if word not in words_to_ignore:
            histogram[word] = histogram.get(word, 0) + 1

    # return only top `histogram_size` most common words (and frequency)
    to_return = {}
    for cnt, word in enumerate(sorted(histogram, key=histogram.get, reverse=True)):
        to_return[word] = histogram[word]
        if cnt = histogram_size-1:
            break

    return to_return
```

 

At first glance, it looks strange (you might say unpythonic!) and is a bit harder to wrap your head around. But actually, it’s just a bunch of normal looking python code with two differences. One is an async in front of the get_histogram method definition, and another is an await in front of the asynchronous slow_function_that_waits_for_io function call.

What’s messier is how to run this code:

**\*Running Your Async Code ***

```
import asyncio
from my_module import import get_popular_words
if __name__ == '__main__':
    coroutine = get_popular_words(
        words_to_ignore=set(['the', 'and', 'to']),
        histogram_size=5,
    )
    popular = asyncio.get_event_loop().run_until_complete(coroutine)
    print(popular)
```

 

If you’re using a framework like [aiohttp](http://aiohttp.readthedocs.io/en/stable/) or [sanic](https://github.com/channelcat/sanic) you’ll just follow the default instructions and not have to think about this much. If you dig into the details you’ll have to learn all about coroutines and what’s happening under the hood.

 

## Negatives

While this performance sounds great, what does it cost? We’ve found programming with asyncio to have some downsides.

 

#### HARD TO USE THE INTERPRETER

One of the nice things about python is that you can always fire up the interpreter and step through code easily. For asyncio code, you need to run your code using an event loop. In practice, this usually means adding a bunch of print statements to whatever isn’t working and re-running your code.

 

#### ALL OR NOTHING

It’s hard to just use asyncio for the few method(s) that would most benefit from the performance, since you need to run your code with an event loop. You’ll have to jump into the deep end!

In python3’s early days, there was valid concern about library support. Today, [python3 has overtaken python2 in library support](https://news.ycombinator.com/item?id=11246662). However, asyncio is trickier. For example, the popular requests library [has chosen to not be asyncio compatible](https://github.com/kennethreitz/requests/issues/2801) (note: [aiohttp](http://aiohttp.readthedocs.io/en/stable/) is a great asyncio compatible alternative). You’ll probably need a new database adaptor.

 

#### YOU HAVE TO THINK ABOUT THE EVENT LOOP

Are you awaiting the result of several actions and then performing some action on that data? If so, you should be using [asyncio.gather](https://docs.python.org/3/library/asyncio-task.html#asyncio.gather). Or, perhaps you want to [asyncio.wait](https://docs.python.org/3/library/asyncio-task.html#asyncio.wait) on future completion? Do you want your future to [run_until_complete](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.AbstractEventLoop.run_until_complete) or [run_forever](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.AbstractEventLoop.run_forever)?

Did you forget to put async in front of a function definition that uses await? That will throw an error. Did you forget to put await in front of an asynchronous function? That will return a coroutine that was never invoked, and intentionally not throw an error!

asyncio doesn’t play that nicely with threads, so if you’re using a library that depends on threading (like [Cassandra’s python driver](https://groups.google.com/a/lists.datastax.com/forum/#!topic/python-driver-user/jVktrQCMd3k)) you’re going to have to come up with a workaround.

 

#### EVERYTHING IS HARDER

Libraries are often buggier and less used/documented (though hopefully that will change over time). You’re not going to find as many helpful error messages on Stack Overflow. Something as easy as writing asynchronous unit tests is non-trivial.

There’s also more opportunities to mess up. For example, when we moved one app to asyncio we forgot to move over a few synchronous API calls that were using [the requests library](http://docs.python-requests.org/en/master/). Since we were running this app on a single thread, those ~100ms API calls were making the whole app hang! It took a while to figure out why.

 

## When should I use asyncio?

It depends! With [more python3 library support than python2](https://news.ycombinator.com/item?id=11246662), there’s no good reason to build a new app using python2. It makes sense to use asyncio if your app wastes a lot of cycles waiting on IO, is a good fit for an asynchronous framework (especially websockets), and resource intensive (reduce your server bill).
