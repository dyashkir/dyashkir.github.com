---
layout: post
title: Postgres and Nodejs Setup setup for testing with Mocha
---

{{ page.title }}
================

<p class="meta">March 26 - Toronto</p>

Postgres is an excellent open source RDBMS. Here is a simple approach to have a setup for your API or integration level tests with Mocha.

##Setup

Once you have Node and Postgres running install an excellent Postgres driver [Node-postgres](https://github.com/brianc/node-postgres)

    npm install pg

then Mocha

    npm install mocha


First we need a snapshot of the database you want to test against.

    mkdir test //for your mocha tests
    sqldump <your-db-name> -f test/testdb.sql

Inside your Mocha test file we can setup what do before tests run. It will look something a long the lines of

    var assert = require("assert")
     , http = require('http')
     , should = require('should')
     , request = require('request')
     , exec = require('child_process').exec


    describe("app", function(){
      before(function(done){
        ...
      });
      after(function(done){
        ...
      });
    
      it('should your tests', function(done){
        ..
        done();
      });
    });

Objective is to setup a clean database run tests then drop the database and end inthe beginning state, ready to run tests again

Here is a simple database create and then run our testdb.sql dump

    function prepare_db(next){
      exec('createdb testdb', function(err){
          if (err !== null) {
          console.log('exec error: ' + err);
          }
          exec('psql -d testdb -f test/testdb.sql', function(err){
            if (err !== null) {
              console.log('exec error: ' + err);
            }
            next(err);
            });
          });
    }

So we first create the database with createdb utility, then run our sql dump with exec

Inside the before function all we need to do is

    before(function(done){
      prepare_db(function(err){
        if(err){
          ..
        }
        //do other setup stuff like launching you server etc
      })
    });

Next we make a simple sql script to drop our test database

    drop database testdb;

Then we can run it with exec same as our sql dump

    function clean_db(next){
      exec('psql -d somedb -f test/dropdb.sql', function(err){
        if (err !== null) {
          console.log('exec error: ' + err);
        }
        next();
      });
    }

And then add it to after function

    after(function(done){
      clean_db(function...

Depending on database size the setup step can be not as fast as we wish if we try to use production size database there. For normal constant testing I found using a simpler dev database simple and faster. It is however possible to setup different snapshots to run the tests as background automated process that will not drive you nuts when you make modification and want to run a quick test.
