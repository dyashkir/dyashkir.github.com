---
layout: post
title: Node.js tutorial for Heather Payne Part 1
---

{{ page.title }}
================

<p class="meta">20 Dec 2011 - Toronto</p>

Starting
--------

This is my first ever blog post, triggered by a post [here](http://heatherpayne.ca/admission-my-project-is-too-hard) by Heather Payne. The original problem is explained [here](http://heatherpayne.ca/this-is-what-im-going-to-build). In general I am seriously impressed by what Ladies Learning to Code are doing, so I thought I should throw in some help.

Doing it simple
---------------

Goal is a solution that is as lean as possible. No extra nice things, no useless frameworks, databases, server and all other crap that beginner does not need. So yes I love redis and we could directly interface with Eventbrite api (is that what LLC use for registration?) but this is not needed.

We have a number of CSV files with attendance data they are in the form in the form of:
    name, name, name, name
each file represent an event, these files can be exported from Eventbrite or another registration site.

Our goal: enter name, find if the person has attended an event and how many times, reply with a witty text based on how many times they attended.

Why Node.js? 
------------

First and reason is that most people who look at this probably know Javascript syntax to a certain degree having done some basic coding for web pages, Node.js is using Javascript! If we want to add some dynamism to the page later we will be working with Javascript both in the browser and inside Node.js. Second is that it requires practically no configuration, we are doing everything in the code and that is where we should focus when starting.

There is one issue that some will consider a disadvantage. Node.js is non-blocking, what does that mean? When we are trying to read a file, make request to another website, write to a database or anything that takes a long time we can not stop and wait for it to finish. Our code has to go on while request is being executed. We then write code that handles results of these requests separately.

Technology breakdown
--------------------

Plane install of [Node.js](http://nodejs.org/), we will not use any framework or database. We will write all code in Javascript. I am a firm believer in learning from basics, frameworks are for when you know what you are doing, not before. 

Full documentation [here](http://nodejs.org/docs/v0.6.6/api/index.html)

I do not know what machine Heather is using, but let us assume Mac for now (just because that is what I am typing this thing on).

Getting Node.js to work
--------------------

[Download node](http://nodejs.org/#download)

Install it from the package.

Check that setup works by using hello world from Node.js homepage. Open your favourite editor and copy paste this or even better type it in by hand.

    var http = require('http');
    http.createServer(function (req, res) {
      res.writeHead(200, {'Content-Type': 'text/plain'});
      res.end('Hello World\n');
    }).listen(1337, "127.0.0.1");
    console.log('Server running at http://127.0.0.1:1337/');

Save it with name index.js in a folder you like, best put it right into your home directory, for me it is /Users/dyashkir 

Now scary part, open your terminal window and put this:

    node index.js

If everything is right you will see Server running at http://127.0.0.1:1337/ go to your browser and put that address in should get Hello World printed. You just built your first HTTP server (kindof like Apache!). Every time you change the index.js file go to terminal and press Ctrl-c then run node index.js again. If you get any errors while running the code this is where you will also be able to see them. If you want to print something you can use 
    console.log('something');
in your code, result will also show in this window.

### HTTP server what is it?? (feel free to skip)

What is an HTTP([Hypertext Transfer Protocol](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)) server? Simple explanation is that client is what you run on your machine, phone etc., server is something that exists remotely(or locally on your machine) and fulfills requests that you send to it. When you visit a web page your web browser interacts with a remote computer that is running a web server application. This application is what provides you with the pages you are looking at, responds when you submit web forms etc. We just wrote a very simple web server, it uses Node.js to respond with simple text every time browser or another application contacts it using HTTP.

First page
----------

Start with making a form for our user to put his name into and then be able to submit it to the server. We were able to return hello word to the browser before, why not a chunk of html that will ask him for the name?


Delete the res.writeHead line, we do not want browser to interpret what we send as text. And let us add some really simple html instead

     var page =
       '<!DOCTYPE html>'+
       '<form>' +
       '<input id=userName name=userName placeholder="enter name" required>'+
       '<button type=submit>Do it!</button>'+
       '</form>';

We are just concatenating string to make the html more readable. We made a form and added a field to enter name and then button to submit the form.

To have our server return it to the user we just replace the Hello world string with page variable.

Going to the same URL should now present the form in our browser. Excellent!

Getting user name into our code
-------------------------------

Now we need to get the name that user entered into our code. If you click on submit you will notice that the name becomes part of the URL, neat no? So all we need to do is have our server parse that URL to get the name.

Let us start by explaining Node.js server code somewhat. When we typed 
    http=require('http'); 
web added http server/client module to our code. When we said 
    .createServer 
we literally created an http server. Notice that we passed something to the createServer function. This is a __callback__, a function that will be executed every time request is made to our server,
    .listen 
tell http to start doing stuff and listening on the port for requests (do not worry about it for now). 

Notice that our callback has two parameters: req and res. First describes the request being made to us, second is the response we will provide, this is the response that browser will interpret and diplay to the user. To give something back we called res.end this finishes processing request and returns the value back to the browser. So to figure out what name user typed we need to understand the req better.

Next step is parse (as in break down into manageable pieces) the URL that browser sent us after we hit the button. To do this we will use another Node.js module called _drumroll_ url.

Add

    var url = require('url');

Next let's get our request processed.

inside the callback add

    var parsedUrl = url.parse(req.url, parseQueryString=true);

parseQueryString tells node to parse parameter part of the url (stuff after ?). Next let's get our name out

   var userName = parsedUrl.query.userName;

Let's check that query has name

    if (parsedUrl.query.userName) {
      var userName = parsedUrl.query.userName;
    }

But we really want to see that name, so let's return it back to user.

   res.end(name);

We also need to make sure that rest of the code does not execute so let's move it into the else part of if statement.

Final result for today should look something like this:

    var http = require('http');
    var url = require('url');
    http.createServer(function (req, res) {
  
      var parsedUrl = url.parse(req.url, parseQueryString=true);

      if (parsedUrl.query.userName) {
        var name = parsedUrl.query.userName;
        res.end(name);
      }else{
        var page = 
        '<!DOCTYPE html>'+
        '<form>' +
        '<input id=userName name=userName placeholder="enter name" required>'+
        '<button type=submit>Do it!</button>'+
        '</form>';
    
        res.end(page);
      }
    }).listen(1337, "127.0.0.1");
    console.log('Server running at http://127.0.0.1:1337/');

Typing name and hitting button (or enter) should now return user name. Great job! You built a custom HTTP server! Take a break tomorrow we will learn how to read our CSV(Comma Separated Value), find the user in it etc.

If you have questions or something is broken hit me up me on [twitter](https://twitter.com/#!/dyashkir) or post it in the comment section.
