---
layout: post
title: Nodejs tutorial for Heather Payne Part 1
---

{{ page.title }}
================

<p class="meta">20 Dec 2011 - Toronto</p>

Starting
--------

This is my first ever blog post, triggered by post [here](http://heatherpayne.ca/admission-my-project-is-too-hard) by Heather Payne. The original problem is explained [here](http://heatherpayne.ca/this-is-what-im-going-to-build). In general I am seriosly impressed by what Ladies Learning to Code are doing, so I thought I should throw in some help.

Disclaimer, typing this in vi with no spellchecker, read at your own risk!

Doing it simple
---------------

My goal is a solution that is as lean as possible. No extra nice things, no useless frameworks, databases, server and all other crap that beginner does not need. So yes I love redis and we could directly interface with Eventbrite api (is that what LLC use for registration?) but this is not needed.

We have a CSV file of some sort with attendance data, lets say it is in the form of:
    name,number,name,number

Our goal, enter name, find match in the CSV reply with a witty text based on how many times person attended.

Technology breakdown
--------------------

Barebones install of [nodejs](http://nodejs.org/), we will not use any framework or database. We will write all code in javascript. I am a firm believer in learning from basics, frameworks are for when you know what you are doing, not before.

Full documentation [here](http://nodejs.org/docs/v0.6.6/api/index.html)

I do not know what machine Heather is using, but lets assume Mac for now(just because that is what I am typing this thing on).

Ok lets do it.

Getting node to work
--------------------

[Download node](http://nodejs.org/#download)

Install it from the package.

Lets check that setup works by using hello world from nodejs homepage. Open your favourite editor and copy paste this

    var http = require('http');
    http.createServer(function (req, res) {
      res.writeHead(200, {'Content-Type': 'text/plain'});
      res.end('Hello World\n');
    }).listen(1337, "127.0.0.1");
    console.log('Server running at http://127.0.0.1:1337/');

Save it with name index.js in a folder you like, best put it right into your home directory, for me it is /Users/dyashkir 

Now scary part, open your terminal window and put this:

    node index.js

If everything is right you will see Server running at http://127.0.0.1:1337/ got to your browser and put that address in should get Hello World printed. You just built your first HTTP server (kindof like Apache!). Every time you change the index.js file go to terminal and press Ctrl-c then run node index.js again.

First page
----------

Lets start with making a form for our user to put his name into and then be able to submit. We were able to return hello word to the browser before, why not a chunk of html that will ask him for the name?


Delete the res.writeHead line, we do not want browser to interpret what we send as text after all. And lets add some really simple html instead

     var page = 
       '<form>' +
       '<input id=userName name=userName placeholder="enter name" required>'+
       '<button type=submit>Do it!</button>'+
       '</form>';

We are just concatenating string to make the html more readable. We made a form and added a field to enter name and then button to submit the form.

To have our server return it to the user we just replace the Hello world string with page variable.

Going to the same url should now present the form in our browser. Excellent!

Getting user name into our code
-------------------------------

Ok now we need to get the name that user entered into our code. If you click on submit you will notice that the name becomes part of the URL, neat no? So all we need to do is have our server parse that URL to get the name.

Lets start by explaining nodejs server code somewhat. When we typed 
    http=require('http'); 
web added http server/client module to our code. When we said 
    .createServer 
we literally created an http server. Notice that we passed something to the createServer function. This is a callback, a function that will be executed every time request is made to our server,
    .listen 
tell http to start doing stuff and listenting on the port for requests (do not worrry about it for now). 

Notice that our callback has two parameters: req and res. First describes the request being made to us, second is the response we will provide. To give something back we called res.end this finishes processing request and returns the value back to the browser. So to figure out what name user typed we need to understand the req better.

Next step is parse (as in break down into manageable pieces) the URL that browser sent us after we hit the button. To do this we will use another nodejs module called *drumroll* url.

Lets add

    var url = require('url');

Next lets use get our request processed.

inside the callaback add

    var parsedUrl = url.parse(req.url, parseQueryString=true);

parseQueryString tells node to parse parameter part of the url (stuff after ?). Next lets get our name out

   var userName = parsedUrl.query.userName;

Try and reload the page from start. It crashes, this is because original page has no query we need to handle that. Lets check for it

    if (parsedUrl.query) {
      var userName = parsedUrl.query.userName;
    }

Ok but we really want to see that name, so lets return it back to user.

   res.end(name);

We also need to make sure that rest of the code does not execute so lets move it into the else part of if statement.

Final result for today should look something like this:

    var http = require('http');
    var url = require('url');
    http.createServer(function (req, res) {
  
      var parsedUrl = url.parse(req.url, parseQueryString=true);

      if (parsedUrl.query) {
        var name = parsedUrl.query.userName;
        res.end(name);
      }else{
        var page = 
        '<form>' +
        '<input id=userName name=userName placeholder="enter name" required>'+
        '<button type=submit>Do it!</button>'+
        '</form>';
    
        res.end(page);
      }
    }).listen(1337, "127.0.0.1");
    console.log('Server running at http://127.0.0.1:1337/');

Typing name and hitting button (or enter) should now return user name. Great job! You built a custom HTTP server! Take a break tomorrow we will learn how to read our csv, find the user in it etc

If you have questions or something is broken hit me up me on [twitter](https://twitter.com/#!/dyashkir) oh and follow as well
