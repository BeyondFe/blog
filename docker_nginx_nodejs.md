title: Nginx and Node.js with Docker
date: 2015-08-25
---

I'll be using Docker to create a Nginx web server and a Nodejs server. Nginx will be configured to use the Nodejs server as a proxy to a specfic url (/api for this example). For Nginx, I'll be using the [official image from Docker](https://hub.docker.com/r/library/nginx/). For Nodejs, I'll be creating a custom Docker image.

Full source code for this example [docker-nginx-node-part1](https://github.com/schempy/docker-nginx-node-part1)

<!-- more -->

### Directory Structure
Here is the directory setup I'll be using:
```bash
nodejs/
	Dockerfile
	src/
		package.json
		server.js
web/
	src/
		public/
			index.html
config/
	nginx.conf
```
		
The **nodejs** directory contains a Dockerfile to create a Docker image and a src directory that stores the nodejs server javascript file and package.json file.

The **web** directory contains the static web files used by the Nginx web server.
The **config** directory contains the custom Nginx configuration file I'll be using for creating the Nginx Docker container.

### The Nginx Docker Image
I need to pull the latest Nginx image from DockerHub. Execute the following bash command to do just that:
```
docker pull nginx:latest
```
The Nginx image uses [Debian](https://hub.docker.com/_/debian/) as the base OS. I'll be using that for creating the custom Nodejs Docker image. 


### The Nodejs Docker Image
Create a file named **Dockerfile** in  the **nodejs** directory with the following:
```bash
FROM debian

RUN apt-get update && apt-get install -y \
	curl \
	python \
	make \
	g++

RUN curl -sL https://deb.nodesource.com/setup_0.12 | bash -
RUN apt-get update && apt-get install -y \
	nodejs

COPY ./src /var/www/api

RUN cd /var/www/api; npm install

EXPOSE 5000
CMD ["node", "/var/www/api/server.js"]
```
The Dockerfile will do the following:

* Lines  3-7 installs libraries that will be necessary for building node modules.
* Lines 9-11 installs the latest Node.js and npm versions.
* Line 13 copies the source code inside the Docker image.
* Line 15 installs the dependencies from npm.
* The node.js server will run on port 5000 so we expose that port on line 17.
* Line  18 defines the command to run which will use the node runtime followed by the path to our node.js server.


To build the Docker Nodejs image execute this bash command from the **nodejs** directory:
```bash
docker build -t /example/nodejs .
```

To confirm we have the necessary Docker images execute the following bash command:
```bash
docker images
```
You should see entries for **example/nodejs**, **nginx** and **debian** under the repository heading. 


Now that we have Docker images we'll use them to build our Docker containers.

### The Nodejs Docker Container

To create a Nodejs Docker container execute the following bash command from the project root directory:
```bash
docker run \
	-d \
	--name "nodejs" \
	-p 5000:5000 \
	example/nodejs;
```

Docker will do the following after executing the bash command above:

* Line 2 instructs Docker to run the container in the background.
* Line 3 instructs Docker to name the container **nodejs**. We'll be using the name of this container within the nginx configuration.
* Line 4 instructs Docker to map the Docker host (port 5000) to the Docker container (port 5000).
* Line 5 instructs Docker to use the NodeJs Docker image, example/nodejs, we previously created.

To ensure the Nodejs Docker container is running goto http://localhost:5000/api in your web browser. You should see a page that displays 'Hello World'. You can also execute the following bash command:
```bash
docker ps
```
You'll see the running container with the name **nodejs**.

### The Nginx Docker Container

I'll be using a custom **nginx.conf** file that will be mapped to the containers **/etc/nginx** directory. Here is the server portion which defines the proxy to the node.js server.
```bash
    server {
        listen 80;
        index index.html;
        server_name localhost;
        error_log  /var/log/nginx/error.log;
        access_log /var/log/nginx/access.log;
        root /var/www/public;

        location ~* /api {
            proxy_pass http://nodejs:5000;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
    }
}
```
The configuration will proxy any requests to the url **/api** to the Nodejs Docker container! Note the **proxy_pass** entry, lines 9-16, uses the name of the Nodejs Docker container, nodejs, that was created.

To create the Nginx container execute the following bash command from the project root directory:
```bash
docker run \
	-d  \
	--name "web" \
	-p 8080:80 \
	-v $(pwd)/web/src:/var/www \
	-v $(pwd)/config/nginx.conf:/etc/nginx/nginx.conf \
	--link nodejs:nodejs \
	nginx;
```

Docker will do the following after executing the bash command above:

* Line 2 instructs Docker to run the container in the background.
* Line 3 instructs Docker to name the container **web**.
* Line 4 instructs Docker to map the Docker host (port 8080) to the Docker container (port 80).
* Line 5 instructs Docker to map our local **web/src** directory to the Docker Nginx containers **/var/www** directory.
* Line 6 instructs Docker to map  our local Nginx configuration file into the containers **/etc/nginx** directory.
* Line 7 instructs Docker to reference the Nodejs Docker container, named **nodejs**, from the Nginx Docker container. This will create a entry in the /etc/hosts/ file to the Nginx Docker container.

Confirm the Nginx Docker container is running by executing the following bash command:
```bash
docker ps
```
You should see a running container with the name **web**.

Point your browser to http://localhost:8080. Click on the link labled API and that will take you to the nodejs server! Notice the link does not contain the port 5000. This is because Nginx is configured to use the /api url as a proxy to the nodejs server.

Full source code for this example [docker-nginx-node-part1](https://github.com/schempy/docker-nginx-node-part1)

