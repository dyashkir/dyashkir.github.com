---
layout: post
title: Ode to Redis 
---

{{ page.title }}
================

<p class="meta">Jan 17 2012 - Toronto</p>

Joy
---

[Redis](http://redis.io/) is a key value store on the surface, it is really a data structure server that is blazing fast, uses little memory, easy as hell to use and feels to be without any bloat whatsoever. It has hashes, lists, sets, ordered sets, pubsub, atomic counter, key expirations and they all work and they all are fast, it also probably slices bread, cures cancer and somehow feels minimal.

My current project I started using Redis as my primary data store for user data, communication, authentication, user grouping and everything else. One of the great things it forces you to do is actually think about what is it exactly you are doing with the data and then think about the data structure you want and optimize it to the type of queries that actually have to be fast. And fast they are, really damn fast!

If you have not tried it yet, try you will not be disappointed it is a gold standard of how things should be done.

Just do not use
    keys whatever*
for your normal queries(like selecting subset of something), God drowns a puppy if you do that.
