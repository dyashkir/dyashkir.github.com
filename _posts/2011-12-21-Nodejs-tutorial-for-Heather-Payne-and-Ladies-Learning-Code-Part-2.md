---
layout: post
title: Node.js tutorial for Heather Payne and Ladies Learning Code Part 2
---

{{ page.title }}
================

<p class="meta">21 Dec 2011 - Toronto</p>

Day 2
-----

If you have not read the previous post please check it. To recap we installed Node.js, figured out how to make a simple web form to allow user to enter his name, then managed to retrieve that name and return it back. Today we will deal with issue of reading CSV(coma separated value) files and extracting the information we want from them.


How to deal with our data
-------------------------

We assume we have a number of files, each has list of names, names are those of the people who attended the event described in the file:

    name, name, name...

When our server starts we need to read all of the CSV files we want to use. Next we need to combine them all together. Once we read the files where would we store the data? We want to search by name and then have a counter of how many times that person attended an event. In Javascript we can define an object to store data in this way.Insert these couple lines before http.serverCreate

    var peopleDictionary = {}; //defines and empty object
    peopleDictionary['dmytro yashkir'] = 1; //its a lie!

This syntax associates the string passed in [] with a value (in this case a number), we can then retrieve the value

    //this will print out the value we stored 1
    console.log(peopleDictionary['dmytro yashkir']); 


Getting our files
-----------------

First let us create some sample files for us to play with. Using terminal or Finder make a sub directory csv_data
    mkdir csv_data

then we should make few files and populate them with data, call the files anything you want but give them .csv extension. Each separate line is a different file you should make

    test user, test user 1, test user2, test user4, test user5
    
    test user, test user 1, test user2, test user4

    test user, test user 1, test user2

    test user

Now you have directory with your data files. Time to read them. To do this we will use _fs_ module of nodejs, fs stands for file system. This is the module that you can use to read, write and manipulate files.

We will insert this code before our http.createServer statement. This is important because we want this code to execute only when server starts (we will improve this in a future tutorial). For now this is good enough.

    
    fs.readdir('csv_data', function(err, files) { //read the directory
  
      files.forEach( function (fileName) {        //go through all files in directory
        fs.readFile('csv_data/' + fileName, 'utf-8', function( err, data){ //read
          console.log(data); //print contents of each file
        });
      });
    });

We will go through this step by step. First we need to know what files we have that we can read for data. For this we read the contents of the csv_data directory. Once that is done the callback that we pass will be called with two parameters (ignore errors for now). Parameter that we called files will contain an array (collection) of all the files in that directory. We are half way there!

Next we use .forEach method that all arrays in javascript have. This method/function will call function passed to eat on each element of the array, what this means is that function we pass is called once for every file name.  Next we readFile, this tell node to read all of content of the file (do not worry about 'utf-8' for now). The last parameter is another callback, this one will be called when file is read into memory.

In this last callback we just print out the contents of the file. Rerun node index.js and you should see all of the file contents printed out.

Ok we are close
---------------

Pieces are falling into place. We have user name, we have data from files, we have data structure to store data. Next we put the file reading and our data structure together. Replace console.log with code to parse a csv file and add names into our data structure (peopleDictionary)

     var names = data.split(',');     //separating string in the file based on comas
     names.forEach( function (name) { //execute code for each name
       name = name.trim();            //remove spaces and line breaks
       if (peopleDictionary[name]){   //check if name is already in the dictionary
         peopleDictionary[name]++;    //increment if name is already present
       }else{
         peopleDictionary[name] = 1;  //new name found set counter to 1
       }
    });

First we split the string containing the file using split (, being the delimiter), splitting creates an array of names then we use forEach to execute a function on each element of that array.

We first trim the name to remove spaces and line breaks. Next we need to handle two cases, user being in the structure and not. If user is in the structure *if(peopleDictionary[name])* then we increment the counter using ++ operator in effect adding 1 to the counter of number of times this user has appeared before. In the else part we handle user that we never saw before by setting his counter to 1.

Final stretch
-------------

We now have structure with counters for users and name to play with let's assemble it!

In the callback of our http server let us add the code to check how many times the user appeared in the csv files and return that number to the user, or let him know that she never showed up.

    //let us trim to remove spaces just in case
    //also put everything lower case just like when we handled our csv file
    var name = parsedUrl.query.userName.trim().toLowerCase(); 

    if (peopleDictionary[name]) {  //check if name is in the dictionary
      res.end(name + ' has attended ' + peopleDictionary[name] + ' times');
    }else{
      res.end(name + ' never attended :(');
    }
 
Fire it up! Enter one of our test user name for example test user. Hit submit and we have a result 4!

Conclusion (for now)
--------------------

We now have a custom built nodejs based http server that on startup reads directory with csv files, and then can reply to users as to how many times she attended an event.

I put our small server into a public [github repository](https://github.com/dyashkir/nodejs_tutorial_llc) that you can view or get the code from.

Next step is to have some code to have witty one liners. Should not be too bad. Best to try and do it yourself first. Next week I will cover this, we will also modify out awesome server to be able to serve flat html files (so we do not need to encode our html right in the code). After that we will do some fun AJAX and user side (inside browser) Javascript.

Hopefully this has been helpful, I enjoyed writing this. If there are any problem with code, you want to ask any question or offer suggestions contact me on twitter [@dyashkir](https://twitter.com/#!/dyashkir) or leave comment here.

Happy Holidays!
