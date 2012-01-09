---
layout: post
title: Node.js tutorial Part 3
---

{{ page.title }}
================

<p class="meta">Jan 5 2012 - Toronto</p>

Final part
----------

Last part of the tutorial that uses Node.js to build a simple page to report user attendance of a workshop, data was stored in a CSV file. Only one simple thing left to do. Instead of having our server return the number of times user attended we need to be able to return a clever phrase to the same effect.

First we need to have a structure to store the phrases we need to return. Simplest way is to use associative structure we used for counting user attendance.

    var phrase = [ 
      { limit:0, text: 'Your kind are not welcome here'},
      {limit:1, text: 'Have you tried turning power off and then on?'},
      {limit:2, text: 'lame joke 123'},
      {limit:10, text: 'Go away stalker!'},
      {limit:100000, text: 'something is terribly wrong'}];

We associated the number of time we want user to attend the workshop before she gets a phrase. Next we need to find which phrase applies to the user, to do this we will loop through the data structure and check if user attendance has exceeded the limit needed.

    //first lets retrieve the number of times user attended
    var attendanceNumber = 0;
    if (peopleDictionary[name]) {  //check if name is in the dictionary
      attendanceNumber = peopleDictionary[name];
    }

    var index = 0; //start with the very first phrase
    do {
      index++;     //increment by 1
    }while ( phrase[index].limit <= attendanceNumber );//check that user 
                                                       //has attended enough

At the end of this loop index will be at the next element after the one we want, so to return the desired phrase we just need to look at the previous element.

    res.send(phrase[index-1].text);

Let us test it out. Run the server then try entering

    test user5

Should get the turn on and off phrase

    test user

Should get lame joke 123 phrase.

Final code can be seen in this repository [https://github.com/dyashkir/nodejs_tutorial_llc](https://github.com/dyashkir/nodejs_tutorial_llc) 

This should cover all of the basic requirements that we had, a lot of things here are not pretty, but they do work and amount of code is manageable with no configuration required and the only installation required is Nodejs.

I am planning to continue writing Nodejs tutorials, perhaps expanding on this problem or playing with something else. Topics I am considering are loading static html instead of hardcoding, connecting to Eventbrite service directly, showing how to deploy this on nodejitsu or heroku. If you have any ideas or request let me know. Cheers!
