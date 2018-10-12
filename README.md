# VaporDockerDemo

A simple project showcasing how Swift [Vapor](https://vapor.codes) can be used with [Docker](https://www.docker.com/). 

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

Hit `return` once more and your terminal prompt should now start with something like `root@<container_id>:/app#`, which reflects the fact that you're now running a bash instance inside of that container.

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

Notice the `-b 0.0.0.0` option passed to the `serve` command. We need to specify the `0.0.0.0` IP address since that's the one that the Docker container is running on (via the output of the `docker ps` command that we ran previously). By default, a `swift run` command will start the server on `localhost`, which is not what we want.

## Testing Things Out

Because of the way the `docker-compose.yml` file is configured, port `8080` inside of the container is mapped to port `8080` on our local machine. This means that you should be able to open up your favorite browser and navigate to `http://localhost:8080/hello` to see the default `"Hello, world!"` output of the Vapor app. Navigating to `http://0.0.0.0:8080/hello` will also work.
