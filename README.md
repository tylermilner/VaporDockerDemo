# VaporDockerDemo

A simple project showcasing how the [Vapor](https://vapor.codes) server-side Swift framework can be used with [Docker](https://www.docker.com/) all the way from development to production deployment.

## Docker Overview

I won't go into too much detail about what Docker is, but essentially it allows you to create a lightweight "container" that packages up everything you need to run your server app. The main advantage of containerization is the ability to have complete control over the environment in which your app will run. This allows your development environment to mimic what will be used in production.

### Dockerfile

The [Dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) specifies instructions for building a Docker image. We'll be creating both a development and production version of this file.

### Docker Compose File

The [Docker Compose file](https://docs.docker.com/compose/compose-file/) defines how our Docker containers are setup and operate together. We'll also be creating both a development and production version of this file.

### Common Docker Commands

#### `docker ps`

The `ps` command lists all currently running Docker containers. We'll use it to check which containers are running so that we can attach to them when needed. When a container is running, the output of this command looks something like:

```bash
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
7c61d587b9f8        api:dev             "bash"              About an hour ago   Up About an hour    0.0.0.0:8080->8080/tcp   vapordockerdemo_api_1
```

#### `docker attach`

The `attach` command will attach the current terminal instance to the specified container. If the container includes an entrypoint like `bash`, then it will allow us to execute terminal commands _inside_ of the running container.

#### `docker stop`

The `stop` command will stop the specified container. You'll need to use when you want to stop containers that are running in the background.

#### `docker build`

The `build` command will take a `Dockerfile` as input to generate a Docker container image. We'll be using this command to generate our production server image.

#### `docker-compose up --build`

The `docker-compose up` command will use a Docker Compose file to create and start a group of containers. We'll be using this command to launch our container ecosystem during both the development and production process.

## Development vs Production Docker Configuration

Using Docker during the development process differs from how you might use it to deploy your production app. The main difference is that your server app won't automatically start when the container is launched. This allows you to be a little more "hands on" during the development process by attaching to the server app's container instance and executing compilation and run commands inside of the Docker container when necessary. A production setup would automatically launch your server app's binary when the container is started.

## The Server App

The Vapor Swift server app in this repo is just the standard [Vapor API template app](https://github.com/vapor/api-template), which is a simple backend that stores Todo list items in a database. The source code has been modified slightly to use a PostgreSQL database rather than the default SQLite database, but all other functionality remains the same. The following endpoints are available when the app is running:

* `GET /`
* `GET /hello`
* `GET /todos`
* `POST /todos`
* `DELETE /todos`

## Getting Started

### Installing Docker

In order to get started, you'll need to install Docker. Since I'm on macOS, I'm using the [Docker Community Edition for Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac).

## Developing with Docker

### Development Dockerfile

 Currently, `Dockerfile-dev` only specifies the latest [Swift Docker image](https://hub.docker.com/_/swift/) as its base image. This is fine for our development environment since we just need something that can build and run Swift apps.

### Development Docker Compose File

 `docker-compose.yml` currently specifies an "api" and a "db" container, which corresponds to our Vapor server app and PostgreSQL database, respectively. The file also maps directories and ports inside of the container to directories and ports on our host machine so that we can share the same development folder on the filesystem and easily test the app from our host machine.

### Running the App

#### Starting the Docker Containers

Once Docker is installed and running, navigate to the project directory and run the following command to start up the containers:

```bash
docker-compose --file docker-compose-dev.yml up --build
```

#### Attaching to the Vapor App Container

In a separate terminal window, execute the following command to list the running containers:

```bash
docker ps
```

Take note of the `CONTAINER ID` for the `api:dev` image, which is the container that will be running the Vapor app. 

Attach to the Vapor app container by executing the following command, replacing `<container_id>` with the identifier output from the command above:

```bash
docker attach <container_id>
```

The command will appear to stall or not do anything, but just hit `return` again and your terminal prompt should now start with something like `root@<container_id>:/app#`, which reflects the fact that you're now running as a bash instance inside of that container.

Because of the way the development Docker Compose file is configured, your project directory on your local filesystem is actually mounted as a volume inside of the container. If you execute the `ls` command inside of your Docker container's terminal instance, you should see the contents of the project directory.

#### Building the Vapor App

To build the Swift Vapor app, execute the following commands in the terminal window that's attached to the Docker container:

```bash
swift build
```

#### Running the Vapor App

Once the build has finished, execute the following command to start the Vapor app inside of the container:

```bash
swift run Run serve -b 0.0.0.0
```

The Vapor app should then startup and output the following to your terminal window:

```bash
Server starting on http://0.0.0.0:8080
```

Notice the `-b 0.0.0.0` option passed to the `serve` command. We need to specify the `0.0.0.0` IP address since that's the one that's been assigned to the Docker container (via the output of the `docker ps` command that we ran previously). By default, a `swift run` command will start the server on `localhost`, which is not what we want (i.e. the app will not be accessible from our host machine).

### Testing Things Out

#### Browser

A quick test can be performed in your web browser to verify the server app is working properly.

Because of the way the `docker-compose.yml` file is configured, port `8080` inside of the container is mapped to port `8080` on our local machine. This means that you should be able to open up your favorite browser and navigate to `http://localhost:8080/hello` to see the default `"Hello, world!"` output of the Vapor app. Navigating to `http://0.0.0.0:8080/hello` will also work.

#### Postman

In order to test the app's integration with the PostgreSQL database, you need to be able to execute `POST` and `DELETE` requests in additon to standard `GET` requests. Inside of the `Postman` directory, a [Postman](https://www.getpostman.com/) collection has been provided that contains these requests to create, delete, and return TODO items. Import the `VaporDockerDemo.postman_collection.json` collection and `VaporDockerDemo-Dev.postman_environment.json` environment into your Postman app and execute the provided requests. Note that an item identifier must be provided in the path of the "Delete TODO" request. By default, it will delete the first item so the request will only work once, assuming an item already exists.

## Deploying with Docker

### Production Dockerfile

`Dockerfile-prod` uses a [multi-stage build process](https://docs.docker.com/develop/develop-images/multistage-build/) to create a standalone Docker container image that contains our production server application. The first stage simply builds the release version of the server app, creating an application binary called `Run`. The second stage builds upon a stock Ubuntu docker image, adding the necessary dependencies for running the binary produced in the first stage and specifies the container entrypoint, or command that gets run when the image is launched.

#### Building the Docker Image

Docker images are tagged in the format `name:version`. Execute the following command to build the Docker image from the production `Dockerfile`:

```bash
docker build --file Dockerfile-prod --tag vapordockerdemo:0.0.1 .
```

#### Running the Docker Image

Once the image is built, execute the following command to start it up:

```bash
docker run --publish 8080:8080 vapordockerdemo:0.0.1
```

The image will run, but the server application will immediately crash on launch with a message like the following:

```bash
Fatal error: Error raised at top level: NIO.ChannelError.connectFailed(NIO.NIOConnectionError(host: "db", port: 5432, dnsAError: Optional(NIO.SocketAddressError.unknown(host: "db", port: 5432)), dnsAAAAError: Optional(NIO.SocketAddressError.unknown(host: "db", port: 5432)), connectionErrors: [])): file /home/buildnode/jenkins/workspace/oss-swift-4.2-package-linux-ubuntu-16_04/swift/stdlib/public/core/ErrorType.swift, line 191
```

The app crashes because it can't initiate a connection to the PostgreSQL database. That's expected at this stage because we haven't yet created a production Docker Compose file that specifies the other containers that need to be launched with our server app (i.e. the PostgreSQL container).

### Production Docker Compose File

In order to stand up all the necessary containers at once, we'll again use a Docker Compose file. The production Compose file is very similar to the one that's used for development, but differs in the following ways:

* The "api" container no longer includes a build step. Instead, it makes use of the `vapordockerdemo:0.0.1` image that we built using the production `Dockerfile`.
* The "api" container is no longer concerned with mounting the project directory on the host machine to the `/app` directory inside of the container. Likewise, a bash prompt is no longer the entrypoint for the container.
* The "api" container no longer maps port `8080` on the host machine to port `8080` of the container. Instead, port `80` on the host machine gets mapped to port `8080` on the container. This more accurately reflects what you would typically expect for a production application - HTTP traffic being served over port `80`.

In the end, the only pieces of data that we need to define our production "api" container are the Docker image to use, the port mapping, and any environment variables.

### Starting the Docker Containers

Again, the `docker-compose` command is used to launch the containers. However, this time we need to specify that we want to use the production compose file:

```bash
docker-compose --file docker-compose-prod.yml up --build
```

### Testing Things Out

Once the containers are running, you should be able to follow the same steps above to test the server application using a web browser or HTTP client like Postman. Note that you will need to use port `80` instead of port `8080` this time, navigating to `http://localhost/hello` in your web browser or using the `VaporDockerDemo-Prod.postman_environment.json` Postman environment.

## References

The content in this repo is based on [this excellent article by bygri](https://bygri.github.io/2018/05/14/developing-deploying-vapor-docker.html).

## WIP

* Security
    * Incorporate suggestions about running Docker apps as non-root user from [this blog post](https://americanexpress.io/do-not-run-dockerized-applications-as-root/).
