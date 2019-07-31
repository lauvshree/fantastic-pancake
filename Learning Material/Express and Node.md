## Lecture ✍️

### What is MERN Stack
**MERN** stack is the name given to a set of JavaScript based technologies used in developing web applications. MERN is the acronym name given to the set of technologies including Mongo DB, Express JS, React JS/ Redux and Node JS. 

In the next few lessons we will be developing the server-side web application for our MERN stack from the scratch.

#### Prerequisites
You should be comfortable with java script coding to follow along. 


We are going to use node.js which helps you write server side code and we will use Express which helps scale up our web applications easily.

In the previous examples we worked out how to *fetch* from a remote server and process the response. We used a web API provided by a web application running on a remote server. Now we will look at how we can create such a web API using node.js and Express.js.

It is presumed that you have node.js installed in your system. As the first step to creating a server side application we will create a new directory called *api*, traverse to that directory in your terminal and run the following command. 

`npm init`

This will init the api directory to serve as a web application. Follow the 
prompts on the screen to complete the intialization. 
* The package name by default is the name of the current folder (*api* in this case). You can specify a different name if you want. 
* Next it asks you for the version you want to set. The default is 1.0.0 and it is recommended you go with the default suggestion. 
* Next it prompts for a description where you can give a short description of what the api intends to do. We are trying to serve the pokemon requests. So the descrtion can be something like *An api to serve Pokemon requests*. 
* Next we specify the entry point into the API, which by default is index.js. For conveniecne of use, we will make *api.js* as the entry point. 
* We will skip the next few prompts (or feel free to fill them in if you fancy it). 
* When it prompts for the author, you can give your name so that you can claim the rights and credits to the this magical API you are about to create.
* License by default is ISC (*Internet Systems Consortium*) which means it is a permissive license that lets people do anything with your code with proper attribution and without warranty.
* It will generate the contents for your package.json and asks you to check if the details are OK.
* Once you confirm, the details are all written on to the package.json. 

Open the api folder now in VSCode and you will see the directory structure with all the content, including the update package.json. 

The next thing we need to create Web API is express. We will install express through the terminal with the following command. 

`npm install express --save`

This will fetch express module from the internet through the npm and install it on your system. If you list your current directory on your terminal, you will see that there is a new folder named 'node-modules' created which contains a whole suite of modules you need which express needs. 

We will now set off on our journey to create the Web API. You may recollect that you mentioned that the entry point for your application is api.js (or anything else that you specified as entry point). Let's open VSCode and create *api.js* in the *api* folder and start writing the code.

```
//This will search for the module express in the node-modules and import it in for use in this appliction

const express = require('express');

//Create an app which an instance of express
const app = new express();

//Create a port on which you want the server to listen. 
//The server is your own system localhost or 127.0.0.1
const port = 3333;

/*  Set up new end point to the default /. 
    This will run the callback with request and response parameters mentioned 
    when the user accesses the path '/'*/


app.get("/", (req,res) => {
    return res.send("Hello World!");
});

//start the server and listen to port number as initialized and 
//invoke the callback

app.listen(port, () => {
    console.log(`listening at http://localhost:${port}`)
})
```

Save and run the javascript and it will start the server and start listening on port *3333*. Now in your browser navigate to http://localhost:3333 or http://127.0.0.1:3333 

You should see *'Hello World!'* rendered. 

Let's add another end point for our server. End-point as you may already know is the access point to an API call on the server. It is called a Web API or REST API. The link we used to access the pokemon API was an end-point. What we created above is also an end point in our server. Now we will add another one.

```
/*This will add end point /test to our server which can be accessed through http://localhost:3333/test*/

app.get("/test", (req,res) => {
    return res.send("From inside test!");
});

```
Save, stop the server (api.js) if it is already running and run the javascript and it will start the server again. Now in your browser navigate to http://localhost:3333/test. See what you get!

But it can be very frustrating to stop and start the server everytime you make changes. There is a package that comes handy in this case. The package is called *nodemon*. Every time you make changes in the server API, it will automatically restart the server. Let's install that in the same directory where we created our api.js. We will install and store it as a dev dependency with the *--save-dev* option because we want to use this only when we are running the server locally in our development environment. 

`npm install --save-dev nodemon`

Once the installation is complete, you will see that the package.json file in the directory is changed to include this devDependency. 

*"devDependencies": {*
    *"nodemon": "^1.19.0"*
*}*

The ^ symbol in front of the 1 denotes that the devDependencies will auto update for all the version starting with 1. eg., 1.20.x, 1.21.x and so on.
Once nodemon is installed, we will make changes to package.json to make use of this and re-start the script when there are changes. We will include the *"start" : "nodemon api.js"* in the scripts section of our package.json. With the changes, the package.json will look like this. 

```
{
  "name": "api",
  "version": "1.0.0",
  "description": "",
  "main": "api.js",
  "scripts": {
    "start" : "nodemon api.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.16.4"
  },
  "devDependencies": {
    "nodemon": "^1.19.0"
  }
}
```

At the command prompt now run `npm start` to start the web server. 
Now make some change to what the end-point *http://localhost:3333/test* returns and see if the server is restarting and changes are reflecting without having to explicitly restart. Magic!

We will look at creating our own Pokemon API in the next lesson.
