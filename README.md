# VaporDockerDemo

A simple project showcasing how the [Vapor](https://vapor.codes) server-side Swift framework can be used with [Docker](https://www.docker.com/).

## Docker Configuration

This repository in its current state reflects the development environment configuration that you might use with Docker. This means that when launching the Docker containers, the server app is not started by default. Instead, this setup relies on you to manually attach to the server app's container instance and execute compilation and run commands. A production setup would automatically build and launch your server app when the container is started.

### Dockerfile-dev

The [Dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) specifies instructions for building our Docker image. Currently, it only specifies the latest [Swift Docker image](https://hub.docker.com/_/swift/) as its base image.

### docker-compose.yml

The [Docker Compose file](https://docs.docker.com/compose/compose-file/) defines how our Docker containers are setup. It currently specifies an "api" and a "db" container corresponding to our Vapor server app and PostgreSQL database, respectively. The file also maps directories and ports inside of the container to directories and ports on our host machine so that we can share the same development folder on the filesystem and easily test the app from our host machine.

## Server App

The Vapor Swift server app in this repo is just the standard [Vapor API template app](https://github.com/vapor/api-template), which is a simple backend that stores Todo list items in a database. The source code has been slightly modified to use a PostgreSQL database rather than the default SQLite database, but all other functionality remains the same. The following endpoints are available when the app is running:

* `GET /`
* `GET /hello`
* `GET /todos`
* `POST /todos`
* `DELETE /todos`

## Getting Started

### Installing Docker

In order to get started, you'll need to install Docker. Since I'm on macOS, I'm using the [Docker Community Edition for Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac).

## Running the App

### Starting the Docker Containers

Once Docker is installed and running, navigate to the directory for this project and run the following command to start up the containers:

```bash
docker-compose up --build
```

### Attaching to the Vapor App Container

In a separate terminal window, execute the following command to list the running containers:

```bash
docker ps
```

This will give you a list of containers that are currently running. The output should look something like:

```bash
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
7c61d587b9f8        api:dev             "bash"              About an hour ago   Up About an hour    0.0.0.0:8080->8080/tcp   vapordockerdemo_api_1
```

Take note of the `CONTAINER ID` for the `api:dev` image, which is the contianer that will be running the Vapor app.

Attach to the vapor app container by executing the following command, replacing `<container_id>` with the identifier output from the command above:

```bash
docker attach <container_id>
```

The command will appear to stall or not do anything, but just hit `return` again and your terminal prompt should now start with something like `root@<container_id>:/app#`, which reflects the fact that you're now running as a bash instance inside of that container.

### Building the Vapor App

Because of the way the `docker-compose.yml` file is configured, your file local filesystem is actually mounted as a volume inside the container. If you execute the `ls` command inside of your Docker container terminal instance, you should see the contents of the project directory.

To build the Swift Vapor app, execute the following commands in the terminal window that's attached to the Docker container:

```bash
swift build
```

### Running the Vapor App

Once the build has finished, execute the following command to start the Vapor app inside if your Docker container:

```bash
swift run Run serve -b 0.0.0.0
```

You're vapor app should then startup and output the following to your terminal window:

```bash
Server starting on http://0.0.0.0:8080
```

Notice the `-b 0.0.0.0` option passed to the `serve` command. We need to specify the `0.0.0.0` IP address since that's the one that the Docker container is running on (via the output of the `docker ps` command that we ran previously). By default, a `swift run` command will start the server on `localhost`, which is not what we want (i.e. the app will not be accessible from our host machine).

## Testing Things Out

## Browser

A quick test can be performed in your web browser to verify the server app is up-and-running.

Because of the way the `docker-compose.yml` file is configured, port `8080` inside of the container is mapped to port `8080` on our local machine. This means that you should be able to open up your favorite browser and navigate to `http://localhost:8080/hello` to see the default `"Hello, world!"` output of the Vapor app. Navigating to `http://0.0.0.0:8080/hello` will also work.

## Postman

In order to test the app's connection to the PostgreSQL database, you need to be able to execute `POST` and `DELETE` requests to the server app in additon to standard `GET` requests. A [Postman](https://www.getpostman.com/) collection has been provided that contains these requests to create, delete, and return TODO items. Import the `VaporDockerDemo.postman_collection.json` Postman collection into your Postman app and execute the provided requests. Note that an item identifier must be provided in the path of the "Delete TODO" request. By default, it will delete the first item so the request will only work once, as long as an item has been created.

## References

The content in this repo is based on this [excellent article by bygri](https://bygri.github.io/2018/05/14/developing-deploying-vapor-docker.html).
