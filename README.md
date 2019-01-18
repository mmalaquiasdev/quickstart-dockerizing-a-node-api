# Dockerizing a Node API

The goal of this project is training how to get a Node.js application into a Docker container.

This guide assumes that you have a working Docker installation and a basic understanding of how a Node.js application is structured.

## Creating the Node.js app

We need create a directory to save all files of our project. So:

```shell
$ mkdir study-dockerizing-a-node-api
$ cd study-dockerizing-a-node-api
```

After create the directory, we need a **package.json** to create a Node.js API:

```shell
$ npm init
```

After that install the express dependencie:

```shell
$ npm i express --save
```

Edit and create a new script to start the node server in the **package.json**:

```json
"start": "node index.js",
```

Now we have a **package.json**:

```json
{
  "name": "study-dockerizing-a-node-api",
  "version": "0.0.1",
  "description": "A repository containing an article and example to start my docker studies",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/mmalaquiasdev/study-dockerizing-a-node-api.git"
  },
  "keywords": [
    "Docker",
    "node",
    "study",
    "api"
  ],
  "author": "Mateus Malaquias",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/mmalaquiasdev/study-dockerizing-a-node-api/issues"
  },
  "homepage": "https://github.com/mmalaquiasdev/study-dockerizing-a-node-api#readme",
  "dependencies": {
    "express": "^4.16.4"
  }
}
```

## Creating a sample API

Now create a **index.js** file that defines a web app using the Express.js framework:

```javascript
const express = require('express');

const PORT = 8080;
const HOST = '0.0.0.0';

const app = express();

app.get('/', (req, res) => {
  res.send('Hello world\n');
});

app.listen(PORT, HOST);

console.log(`Running on http://${HOST}:${PORT}`);
```

## Creating Docker ignore file

Before creating a docker image it's intresting to make a file named .dockerignore in the root directory. When this file exists the CLI modifies the context to exclude files and directories that match patterns in it. This will help us avoid unnecessarily sending large or sensitive files and directories to the image when using ADD or COPY command.

So create the file and edit it:

```shell
$ touch .dockerignore
$ vim .dockerignore
```

Write the content above:

```
node_modules
npm-debug.log
```

This will prevent your local modules and debug logs from being copied into your Docker image.

## Creating a Dockerfile

A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Using docker build users can create an automated build that executes several command-line instructions in succession.

Create an empty file called Dockerfile:

```shell
$ touch Dockerfile
```

Open this file with your favorite text editor. And lets go!

```shell
$ vim Dockerfile
```

### FROM

The first step is define what image we want to use to build our API. In this study i'm using the latest LTS (long term support) version 8 of node available from the Docker Hub.

```Dockerfile
FROM node:8
```

### WORKDIR

We need create a directory inside of the container for hold our application code. For this will be using the struction **WORKDIR**.

```Dockerfile
WORKDIR app
```

If the WORKDIR doesn’t exist, it will be created a generic folder for us. And we can use this instruction in multiple times inside of our Dockerfile.

### NPM

Multiple tutorials in the internet show to us a command to install the NPM inside of our container, but this steps is not required anymore. This image comes with Node.js and NPM already installed so the next thing we need to do is to install your app dependencies using the npm binary.

### COPY

The COPY instruction copies new files or directories from <src> and adds them to the filesystem of the container at the path <dest>.
  
So we need copy the package.json AND package-lock.json, insed of our WORKDIR.

```Dockerfile
COPY package*.json ./
```

After that we use the copy instruction to copy our application source.

```Dockerfile
COPY . .
```

### RUN

The RUN instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile.

So use this command to install the project dependencies.

```Dockerfile
RUN npm install
```

### EXPOSE

The EXPOSE instruction informs Docker that the container listens on the specified network ports at runtime. You can specify whether the port listens on TCP or UDP, and the default is TCP if the protocol is not specified.

The EXPOSE instruction does not actually publish the port. It functions as a type of documentation between the person who builds the image and the person who runs the container, about which ports are intended to be published. To actually publish the port when running the container, use the -p flag on docker run to publish and map one or more ports, or the -P flag to publish all exposed ports and map them to high-order ports.

```Dockerfile
EXPOSE 8080
```

### CMD

The main purpose of a CMD is to provide defaults for an executing container. Here we will use the basic npm start.

```Dockerfile
CMD [ "npm", "start" ]
```

### Your Dockerfile should now look like this:

```Dockerfile
# Node runtime version
FROM node:8

# The api directory insede of the docker image
WORKDIR /app

# A wildcard is used to ensure both package.json AND package-lock.json are copied to the docker image
COPY package*.json ./

# Copy the project to the docker image
COPY . .

# Install the API dependencies with NPM
RUN npm install

# Informe to the docker, the door let's use
EXPOSE 8080

CMD ["npm", "start"]
```

## Building your image

Go to the directory that has your Dockerfile and run the following command to build the Docker image.

```shell
$ docker build -t <your username>/node-hello-world .
```

## Run the image

Running your image with -d runs the container in detached mode, leaving the container running in the background. The -p flag redirects a public port to a private port inside the container.

```shell
$ docker run -p 8080:8080 -d <your username>/node-web-app
```

## Test

Open your web browser or postman application and make a request to the URL above:

```
http://localhost:8080
```

Or use the curl application inside the terminal.

```shell
$ curl -i localhost:49160
```

You need see a simple Hello world message.

I hope this tutorial helped you get up and running a simple Node.js application on Docker.

## References

- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#dockerignore-file)
- [Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)
- [Trying Docker for the First Time 📖 Many Learnings!](https://www.youtube.com/watch?v=2tvbkCW4OIY)

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Contributors

See also the list of [contributors](https://github.com/mmalaquiasdev/study-dockerizing-a-node-api/graphs/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
