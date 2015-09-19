title: Node.js and Docker Using Host Volumes
date: 2015-09-17
---

A previous post, [Nginx and Node.js with Docker](http://schempy.com/2015/08/25/docker_nginx_nodejs/), I created a Node.js Docker container that copied source files inside a Docker image. It's a challenge to change any of the files in the Node.js Docker container since it's baked into the Docker image. A different approach would be to mount a host folder in the Docker container. This way I can edit/add/delete files which would be reflected in the Docker container. The idea is to create a bare Node.js server that I can conintue to develop.

Full source code for this example [docker-nodejs-unbaked](https://github.com/schempy/docker-nodejs-unbaked)

<!-- more -->

### Game Plan

1. Create a Dockerfile for Node.js. This Dockerfile will do the following:
	* Install Node.js
	* Install [pm2](https://github.com/Unitech/pm2) which is a process manager for a Node.js app.
	* Add a  volume to expose a folder created by the Docker image. This folder is where my host folder will be mounted.
	* Add a bash shell script that is responsible for starting the Node.js server.
2. Create a bash shell script that wil start the Node.js server.
3. Create the Node.js file that will start a Node.js server.
4. Build the Docker image and start the Docker container with that image.

### Directory Structure
```bash
api/
	package.json
	server/
		index.js
		routes/
			widget.js
docker/
	Dockerfile
	start.sh
```
The api/server directory will contain the entry point for our Node.js application and the api/server/routes directory contains the application end-point routes.
The docker directory will contain the Dockerfile to create the Docker image and a script that the Docker image will run.

### Step 1: Node.js Dockerfile
Here is the Dockerfile:
```bash
FROM phusion/baseimage

RUN apt-get update && apt-get install -y \
        curl \
        python \
        make \
        g++
RUN curl -sL https://deb.nodesource.com/setup_0.12 | bash -
RUN apt-get update && apt-get install -y \
        nodejs

RUN npm install pm2 -g

VOLUME ["/var/www/example.com/api"]
ADD start.sh /start.sh
RUN chmod 755 /start.sh
CMD ["/start.sh"]
```
Notice the **VOLUME** instruction, line 14, which will create the directory in the container. Also the **ADD** instruction, line 15, that will add the start.sh file to the container. 

### Step 2: Create the start.sh script

```bash
#!/bin/bash

if [ -z "$NODE_ENV" ]; then
    export NODE_ENV=development
fi

cd /var/www/example.com/api
npm install
cd /var/www/example.com/api/server
pm2 start -x $APP --name="app" --no-daemon --watch
```

This script will do the following:

* Lines 3-5 set the environmental variable **NODE_ENV** to development if the variable was not passed as a parameter from the docker command (more on this in Step 4).
* Lines 7-8 change to the directory that will contain a **package.json** file so we can use npm install for the dependencies.
* Line 9-10 change to the directory that will contain the Node.js server starting point. Use [pm2](https://github.com/Unitech/pm2) to start our application. The name of the Node.js starting point file will be passed as a parameter from the docker command. A neat feature of [pm2](https://github.com/Unitech/pm2)  is the [watch](http://pm2.keymetrics.io/docs/usage/watch-and-restart/) parameter which will restart the Node.js server if a file has changed!

### Step 3: Create the Node.js server entry point file
```javascript
var express = require('express');
var app = express();
var bodyParser = require('body-parser');
var port = 5000;

// Support json encoded bodies.
app.use(bodyParser.json());

// Parse application/x-www-form-urlencoded
app.use(bodyParser.urlencoded({ extended: false }));

// Routing middleware.
app.use('/api/widget', require('./routes/widget'));

// Error handling middleware must be after all other middleware and routing.
// Handle error in development mode.
if (app.get('env') === 'development') {
    console.log('running in dev mode');
    app.use(function (err, req, res, next) {
        res.status(500).json(err.stack);
    });

// Handle error in production mode.
} else {
    console.log('running in production mode');
    app.use(function (err, req, res, next) {
        res.status(500).json(err.message);
    });
}

app.listen(port, function () {
    console.log('Listening server on port ' + port);
});
```

I'm using [express](https://github.com/strongloop/express) for the server. This is a pretty basic Node.js server setup. I'm setting a specific route for the URL /api/widget on line 13. Here is the route:

```javascript
var express = require('express');
var router = express.Router();
var _ = require('lodash');
var widgets = [
	{ id: 1, name: 'Icons' },
	{ id: 2, name: 'Buttons' },
	{ id: 3, name: 'Scroll Bars' }
];

// Get all widgets.
router.get('/', function(req, res, next) {
	res.status(200).json(widgets);
});

// Get a specific widget.
router.get('/:id', function(req, res, next) {

	// Find specific widget.
	var widget = _(widgets)
		.find(function(value) {
			return value.id	== req.params.id;		
		});

	// If widget was not found set as empty object.
	if (_.isEmpty(widget)) {
		widget = {};
	}

	res.status(200).json(widget);
});

module.exports = router;
```
I'm only defining a HTTP GET request for all widgets and a specific widget.

### Step 4: Build the Docker image and container

Change to the /docker directory and execute the following bash command:
```bash
docker build -t example/nodejs .
```
This will build the Dockerfile. To confirm this successfully build the image execute the following bash command:
```bash
docker images
```
You'll see **example/nodejs** under the **REPOSITORY** heading.

Now we can start a container based on this Docker image. Change to the directory /api and execute the following bash command:
```bash
	docker run  \
		-d \
		--name "example_api" \
		-p 5000:5000 \
		-e "APP=index.js" \
		-e "NODE_ENV=development" \
		-v $(pwd)/api:/var/www/example.com/api \
		example/nodejs;
```

Docker will do the following after executing the bash command above:

* Line 2 instructs Docker to run the container in the background.
* Line 3 instructs Docker to name the container example_api.
* Line 4 instructs Docker to map the Docker host (port 5000) to the Docker container (port 5000).
* Lines 5-6 instructs Docker to set the environmental variable APP to index.js and NODE_ENV to development.
* Line 7 instructs Docker to map the local api directory into the containers /var/www/example.com/api directory.
* Line 8 instructs Docker to use the Docker image, example/nodejs, we previously created.

To confirm this ran successfully execute the following bash command:
```bash
docker ps
```
You'll see **example_api** under the **NAMES** heading.
Now you can goto a browser to test out the URL's. 
To get all the widgets goto http://localhost:5000/api/widget
To get a specific widget just add the ID of a widget to the URL http://localhost:5000/api/widget/1

Try changing any of the files under /api/server and [pm2](https://github.com/Unitech/pm2)  will restart the Node.js server! You can even add more routes!

Full source code for this example [docker-nodejs-unbaked](https://github.com/schempy/docker-nodejs-unbaked)

