---
layout: post
title: Starting with Redis tutorial
---

{{ page.title }}
================

<p class="meta">Jan 24 2012 - Toronto</p>

Install
-------

Download and run it: [Redis](http://redis.io/download)

Once the server is running you can use command line Redis utility to connect to it. 

    redis-cli

by default if will connect to the local Redis instance

Let's build something
---------------------

We are going to use Redis to build a simple data model for storing user data, authenticating users, storing sessions, grouping users. 

Storing user data
-----------------

First we probably want to have our users have unique IDs. If we have multiple clients accessing and creating users how to ensure that this id is unique. Simple approach is to use

    incr current_user_id

this operation is atomic, so each client process will always get a different user id.

User data can be stored in a number of ways in Redis. The most obvious is to store user data in JSON string under a key.

    set user_1 '{"name":"Pete"}'
    set user_2 '{"name":"Pete"}'

    get user_1

this approach is fine, however there are certain improvement that can be made. Each key in Redis can have an expiration timer, this adds to the overhead. To fix this issue we can store all of our users under a hash, where user ids will be separate entries in the hash.

    hset user 1 '{"name":"Pete"}'
    hset user 2 '{"name":"Bob"}'

    hget user 1

this approach will reduce memory overhead for keys we are storing. It also allows us to retrieve multiple users at the same time by retrieving multiple entries from the hash.

    hmget user 1 2

What if our user object is large?
---------------------------------

However let's consider a relatively large user object. If we store in it in JSON we will need to parse the whole thing every time we retrieve it. If we always need the whole object then this might be acceptable. If the objects are small then it will also work well. But if objects are large and we have many queries where we need only small portion of the object (a pretty common case)?

We can store each user a separate hash

    hset user_1 name Pete
    hset user_1 bday 12938712837
    hset user_1 active YES

    hset user_2 name Bob

    hget user_1 active
    hget user_1 bday

We can also combine multiple hash sets

    hmset user_3 name Pete bday 123131313 active YES

and then get exactly what we want

    hmget user_3 bday active

This approach allows us to use less memory on the client parsing JSON we also can query separate fields which can save us a lot of effort when working with large objects.

Sessions
--------

Let's say we want to have some way for user to login using email as their user id and create a session for them to use from then on. We also want to make this session temporary.

    set session_823712387193 '{"user":"1", "data":"soemthing"}'
    
expire it after 100 minute (time is in seconds)

    expire session_823712387193 6000

if we want session to extend is user is active we just use
    
    expire session_823712387193 6000

again, thus resetting the timer. If for whatever reason we want to get rid of our session

    del session_823712387193

If we do not care about expirations we can store session in session hash. We can also use our user idea of storing each session as a separate hash.

Group those users up!
---------------------

Usually we need to maintain certain groupings of data. For example want to know all paying customers, all customers that have been banned. How can we accomplish this? Redis does not allow us to query value stored under a key. Naive approach to this is encoding this data into the user key.

    set user_1_pay_banned '{"bad_idea":"do not do"}'

then using *keys* command to search this. Do not do this. Redis has to go through ALL of your keys, this is really really slow.

Best way is to maintain sets of users and modify them incrementally. Let's use Redis ordered set to maintain, latest user updates.
    
    zadd update_time 12837123 user_1
    zadd update_time 12837124 user_2
    zadd update_time 12837125 user_3
    zadd update_time 12837126 user_4
    zadd update_time 12837127 user_5

ordered sets require a score for each member, we can use time as this score to have an ordered set of updates. To retrieve 3 last updated users

    zrevrange update_time 0 3

Once we have user ids we can retrieve records for them or in case we used user hash storing method we can retrieve parts of the users we want.

    get user_5
    get user_4
    get user_3

If we are using keys to store users we can combine these gets into a single

    mget user_5 user_4 user_3

Pipelining
----------

Most Redis libraries (like node_redis) pipeline your request by default so there is no need to worry about that.

Conclusion
----------

My current project I am using Redis as the main data store, it surprising just how well this works when you know exactly what has to be queried and can use it's structures to help you. It is also damn fast. My Node.js app can run 2 worker processes with 99% cpu utilisation basically doing nothing else but querying Redis for *zrevrange* and *hmget* and Redis instance barely goes over 5% cpu. This is not a scientific benchmark but feels impressive nonetheless. Have fun with it, Redis has more features to explore as well pubsub, transactions, lists, built in Lua interpreter.
