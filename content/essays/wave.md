---
title: In Defense of Simple Architectures
thumbnail: "fonts.jpg"
featured: true
---

Wave is a $1.7B company with 70 engineers whose product is a CRUD app that adds and subtracts numbers. In keeping with this, our architecture is a standard CRUD app architecture, a Python monolith on top of Postgres.  [Starting with a simple architecture and solving problems in simple ways](https://twitter.com/danluu/status/1462607028585525249)  where possible has allowed us to scale to this size while engineers mostly focus on work that delivers value to users.

Stackoverflow scaled up a monolith to good effect ( [2013 architecture](https://nickcraver.com/blog/2013/11/22/what-it-takes-to-run-stack-overflow/)  /  [2016 architecture](https://nickcraver.com/blog/2016/02/17/stack-overflow-the-architecture-2016-edition/) ), eventually getting acquired for $1.8B. If we look at traffic instead of market cap, Stackoverflow is among the top 100 highest traffic sites on the internet (for many other examples of valuable companies that were built on top of monoliths,  [see the replies to this Twitter thread](https://twitter.com/danluu/status/1498678300163588096) . We don’t have a lot of web traffic because we’re a mobile app, but Alexa still puts our website in the top 75k even though our website is basically just a way for people to find the app and most people don’t even find the app through our website).

> There are some kinds of applications that have demands that would make a simple monolith on top of a boring database a non-starter but, for most kinds of applications, even at top-100 site levels of traffic, computers are fast enough that high-traffic apps can be served with simple architectures, which can generally be created more cheaply and easily than complex architectures.

1. Despite the unreasonable effectiveness of simple architectures
2. Most press goes to complex architectures. 

For example, at a recent generalist tech conference, there were six talks on how to build or deal with side effects of complex, microservice-based, architectures and zero on how one might build out a simple monolith. There were more talks on quantum computing (one) than talks on monoliths (zero). Larger conferences are similar; a recent enterprise-oriented conference in SF had a double-digit number of talks on dealing with the complexity of a sophisticated architecture and zero on how to build a simple monolith. Something that was striking to me the last time I attended that conference is how many attendees who worked at enterprises with low-scale applications that could’ve been built with simple architectures had copied the latest and greatest sophisticated techniques that are popular on the conference circuit and HN.

Our architecture is so simple I’m not even going to bother with an architectural diagram. Instead, I’ll discuss a few boring things we do that help us keep things boring.

We’re currently using boring, synchronous, Python, which means that our server processes block while waiting for I/O, like network requests. We previously tried Eventlet, an async framework that would, in theory, let us get more efficiency out of Python, but ran into so many bugs that we decided the CPU and latency cost of waiting for events wasn’t worth the operational pain we had to take on to deal with Eventlet issues. The are other  [well-known async frameworks for Python](https://twitter.com/mcfunley/status/1194713713330122752) , but users of those at scale often also report  [significant fallout from using those frameworks at scale](https://twitter.com/mcfunley/status/1194715290841432064) . Using synchronous Python is expensive, in the sense that we pay for CPU that does nothing but wait during network requests, but since we’re only handling billions of requests a month (for now), the cost of this is low even when using a slow language, like Python, and paying retail public cloud prices. The cost of our engineering team completely dominates the cost of the systems we operate.

Rather than take on the complexity of making our monolith async we farm out long-running tasks (that we don’t want responses to block on) to a queue.

A place where we can’t be as boring as we’d like is with our on-prem datacenters. When we were operating solely in Senegal and Côte d’Ivoire, we operated fully in the cloud, but as we expand into Uganda (and more countries in the future), we’re having to split our backend and deploy on-prem to comply with local data residency laws and regulations. That’s not exactly a simple operation, but as anyone who’s done the same thing with a complex service-oriented architecture knows, this operation is much simpler than it would’ve been if we had a complex service-oriented architecture.
