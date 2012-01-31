---
layout: post
title: Load testing Node.js app on Heroku with Blitz.io 
---

{{ page.title }}
================

<p class="meta">Jan 31 2012 - Toronto</p>

Setup
-----

If you have tried using Siege to test your app on heroku you probably realized that something is not right. Blitz plugin makes this pretty easy, even a developer can do it.

Most of these steps can be found in the great documentation Heroku has on their site, but few important ones are missing (link)[http://devcenter.heroku.com/articles/blitz] 

Add Blitz.io plugin to your Heroku app by going to your app folder and

    heroku addons:add blitz:250

250 is the free tier, you can test to your heart's content. Higher tiers are billed by second so you can run large test if you need and only get charged few bucks for them. Next install the gem

    gem install blitz

Next figure out your credentials, go to your app page on heroku then to the plugin page from there. If you are a contributor the user name will be the one of the app owner not your own

    --api-key

you will get back the user name and a key

Next setup these credentials

    blitz api:init

Let us try rushing

     blitz curl -r california -p 1-250:60 http://<your app>.herokuapp.com/

If you are using http this will work, for https you need to add a route into your app that tells Blitz.io that it is your app! For Express just throw this in

    app.get('/<the code that you get from console when you try to run your https app>', function (req, res) {
      res.send('42');
    });

Done. Next we can integrate this into our process on continuous deployment (once I have this polished for my own app I will post about it). There are many things you can test as well, different regions (very interesting to see how this works from Singapore and Japan). You can see what happens when you scale your app to multiple Heroku workers as well.
