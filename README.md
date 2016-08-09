# MVC-NodeJS
In Node.js, because we are also creating the webserver too, we need to add three steps in at the beginning:

<h2>
Request			--> CONTROLLER  --> Request  --> MODEL  <br> <br>

MODEL       --> Response    --> CONTROLLER   --> Response   --> VIEW    --> Response
</h2>

Request comes in to server
Server sends request to router
Router sends request to appropriate controller
Looking at this, we can see that we are going to need the following objects (or collections of objects):

Server – to listen to and respond to HTTP requests
Router – to send the incoming requests to the correct controller
Controllers – to perform operations & interrogate the data
Model – to provide the data
Views – to provide the HTML rendering we’re going to see in the browser
Building our framework
To keep things clean, lets create a new folder to put our site in.

Creating the node.js server
Create a file called server.js with the following content:

var http_IP = '127.0.0.1';  <br>
var http_port = 8899;  <br>
var http = require('http'); <br> 
var server = http.createServer(function(req, res) {  <br>
  require('./router').get(req, res); <br>
}); // end server() <br>
server.listen(http_port, http_IP);  <br>
console.log('listening to http://' + http_IP + ':' + http_port);  <br>

The only new thing there, is that rather than outputting anything directly here, we are calling in a ‘router’ module and sending it the request & response object. This makes sense if you look back at the steps the application needs to take: server sends request to router.

<h1>Building the router</h1>

In the same folder as the server.js file, create a new file called router.js and save it with the following content:

var url = require('url');  <br>
var fs = require('fs'); <br>

exports.get = function(req, res) {  <br>
  req.requrl = url.parse(req.url, true); <br>
  var path = req.requrl.pathname;<br>
  if (/.(css)$/.test(path)) {  <br>
    res.writeHead(200, {  <br>
      'Content-Type': 'text/css' <br>
    }); <br>
    fs.readFile(__dirname + path, 'utf8', function(err, data) { <br>
      if (err) throw err; <br>
      res.write(data, 'utf8'); <br>
      res.end(); <br>
    }); <br>
  } else { <br>
    if (path === '/' || path === '/home') { <br>
      require('./controllers/home').get(req, res); <br>
    } else { <br>
      require('./controllers/404').get(req, res); <br>
    } <br>
  } <br>
}; <br>


In the router we are doing a few different things:

Importing the URL and FileSystem (fs) modules that come as part of your node.js install
Exporting a function called “get” – this allows our server file to use this function, passing the request and response objects through
Getting the path of the URL request
Testing to see whether the request is for a CSS file and loading it in if so. There are a number of different ways of handling static files, but this method is fine for what we’re doing here
Sending the request to the correct controller, if it’s not for a CSS file
At this point in our development, it’s number 5 we are interested in. We are checking for two URL paths ‘/’ and ‘/home’ which will route to a controller called ‘home’. We then send any other URL requests to a controller called ’404′. Let’s take a look at these controllers.

<h1>Building the controllers </h1>

Create a folder in the root of your site, called ‘controllers’. Inside this folder create a file called ‘home.js’ with the following content:

var template = require('../views/template-main');   <br>
var test_data = require('../model/test-data');   <br>
exports.get = function(req, res) {   <br>
  var teamlist = test_data.teamlist; <br>
  var strTeam = "", <br>
    i = 0; <br>
  for (i = 0; i < teamlist.count;) { <br>
    strTeam = strTeam + "<li>" + teamlist.teams[i].country + "</li>"; <br>
    i = i + 1; <br>
  } <br>
  strTeam = "<ul>" + strTeam + "</ul>"; <br>
  res.writeHead(200, { <br>
    'Content-Type': 'text/html' <br>
  }); <br>
  res.write(template.build("Test web page on node.js", "Hello there", "<p>The teams in Group " + teamlist.GroupName + " for Euro 2012 are:</p>" + strTeam)); <br>
  res.end(); <br>
}; <br>

Right at the top of this file you can see that we are including the rest of our MVC structure the model and the view. Referring back to the diagram we remember that the controller should do two main things:

The controller (normally) performs some operations using the response from the model
The controller sends a response to the view

<h1>Performing operations on model data</h1>

Our task here is quite simple: get a data object (teamlist) and create a HTML string putting the items into an unordered list. In the next step we’ll see how we create this data.

<h1>Sending a response to the view</h1>

We will use a view (think template) called template-main, which we will create in a few minutes. Note that through the template.build function that we are only sending three content items; within the controller there is no reference to how this content will be displayed. In this example you will notice that we are sending through the HTML for the <ul> list, there are arguments against having even this level of markup in a controller, but we’re going for a simple example here so let’s keep it in!

<h1>Getting data from the model</h1>

In a future post we’ll see how we can extend this demo by getting the data out of a database, but for the sake of remaining focused on the point right now we’ll fake it by creating and returning a JSON object.

So, create a folder in the root of your site called ‘model’. Inside this folder create a new file called ‘test-data.js’ with the following content:

var thelist = function() {   <br>
  var objJson = { <br>
    "GroupName": "D", <br>
    "count": 4, <br>
    "teams": [{ <br>
      "country": "England" <br>
    }, { <br>
      "country": "France" <br>
    }, { <br>
      "country": "Sweden" <br>
    }, { <br>
      "country": "Ukraine" <br>
    }] <br>
  }; <br>
  return objJson; <br>
}; <br>
exports.teamlist = thelist();   <br>


Through the export function we are exposing the function thelist() which itself is returning a JSON object. This is how – for now – we will return a data object to the teamlist in our controller.

<h1>Using a view template to create the HTML page</h1>
Now that we have our controller pulling in some data from the model and doing something with it, we want to show the outcome in a webpage. We’ve seen that we’re going to send the data to a build function in a view template, so let’s create that now.

Create a folder in the root of your site called ‘views’. Inside this folder create a new file called ‘template-main.js’ with the following content:
 <br>
exports.build = function(title, pagetitle, content) {   <br>
  return ['<!doctype html>', <br>
  '<html lang="en">nn<meta charset="utf-8">n<title>{title}</title>', <br>
  '<link rel="stylesheet" href="/assets/style.css" />n', <br>
  '{pagetitle}', <br>
  '<div id="content">{content}</div>nn'] <br>
  .join('n') <br>
  .replace(/{title}/g, title) <br>
  .replace(/{pagetitle}/g, pagetitle) <br>
  .replace(/{content}/g, content); <br>
}; <br>

Here we are directly exposing the build function, which takes three parameters: title, pagetitle and content.

The first thing we are doing here is joining an array of strings, placing a new line ‘n’ between them. This creates one long string of HTML that will render on separate lines leaving the source code more readable. This string is the basis of our template, and contains a number of placeholders for the content items {title}, {pagetitle} and {content}.

Naturally then, the second step here is to replace these placeholders with the actual content sent through via the function arguments. The three replace functions are doing exactly that, running a very simple regular expression to globally change all occurrences found in the string. This means that you can have {title} in several places in your template, and each will get replaced. Note that we are also chaining the join and replace commands; the separate lines are for readability. Seeing as it works when laid out like this there is little sense in putting them all on one line and making the script harder to read.

<h1>Using a static CSS file</h1> <br>
You may have noticed that in the HTML view template that we are referencing a static CSS file; remember in the router how we created a statement checking for the .css file extension? Now is the time to implement it.

In the root of your site create a folder called ‘assets’. Inside this folder create a text file called ‘style.css’ with the following content:
 <br>
* {font-family:arial, sans-serif;}

This isn’t the most complex CSS file in the world, but the change in font is enough for us to know that it is working!
 <br>
<h1>Creating a 404 page</h1>

The final piece of our site is to create our 404 page. We already have a suitable template – our template-main view – so we’ll re-use that. Our router is also already set to send people to a 404 if it doesn’t recognise the URL path requested. So all that remains for us to do is to create the controller.

In your ‘controllers’ folder create a new file called ’404.js’ with the following content:

var template = require('../views/template-main');   <br>
exports.get = function(req, res) {   <br>
  res.writeHead(404, { <br>
    'Content-Type': 'text/html' <br>
  }); <br>
  res.write(template.build("404 - Page not found", "Oh noes, it's a 404", "<p>This isn't the page you're looking for...</p>")); <br>
  res.end(); <br>
}; <br>

You should be able to see what we’re doing here. Using the same template - views/template-main - we send it three items of content. The only difference this time is that we have changed the status code of the response – in the res.writeHead function – from 200 (Ok) to 404 (not found).

<h1>Let’s run the site! </h1> <br>
That’s all the pieces of our MVC framework in place, so let’s try out our site. In terminal cd into the root folder of your site and run it with the command:

node server.js   <br>

Now head to 127.0.0.1:8899 in browser and you should see something like this: 

Let’s test our other path from the router, by navigating to 127.0.0.1:8899/home 

Finally, let’s test our 404 page, by going to a URL that we haven’t added to the router, for example 127.0.0.1:8899/test 

<h1>Success!</h1> <br>
So there we have it. We now have a working website in node.js written in a maintainable and scalable MVC framework. In future posts I will take a look at how to remove some of the pain by using the express.js framework, and also how to get the data from a MongoDB database.

<h2>Get the source code</h2> <br>
https://github.com/cloudaves-sanjay/MVC-NodeJS/
