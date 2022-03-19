# starwars-api-backend-skeleton

---

### Backend API Learning Workflow:

---
### Stage-7:
<span style="color:#FF1B55FF">Deploying our backend</span>

#### Introduction

Up to this point, we have developed our Starwars API backend.
Our backend consists of three servers:
1. Python Flask app to provide the API endpoints
2. MySQL server to store the user information
3. REDIS server to store the tokens for ongoing sessions

We have been running these servers on our local computer, which is a development machine.
In this stage, we will learn about how to deploy our backend in a production server.
There are different environments we can deploy our servers: Amazon Web Services (AWS), Google Cloud Platform (GCP), Heroku, and so on.
These enviroments provide virtual computing resources to run our servers on.
These resources are typically provided as *virtual machines*, with operating systems such as Linux and Windows, on which we install our software.

If your servers are going to serve thousands of requests at the same time, you will eventually need to scale up your servers.
For this, you will need to have multiple virtual machines running the same servers, where requests are distributed over these servers.
One problem there is, how are you going to manually install these servers on each virtual machine?
Note that, you need to install the server software and its dependencies, and configure the server to customize it for your application.
You need to keep repeating this process for different machines, possibly with different operating systems. 

Containerized environments help to solve this problem.
A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings. Please read https://www.docker.com/resources/what-container/ to learn more about Docker containers.

#### Dockerizing our backend

Dockerizing our backend means to prepare a Docker image and configuration for each of our servers: Python Flask server, MySQL server, and REDIS server. Each Docker image represents an individual software with a specific purpose. Thus, we need three Docker images, each of which will be instantiated to a separate Docker container.
Each image will contain the software, the runtime of th software, ie. all the dependencies (libraries and tools) to run the software, and the related configurations. It is a complete packaging of the software, so that we can install and run the image on any machine with Docker installed, without needing any other software.

Fortunately, there are ready-to-use, official images for MySQL server and REDIS, so we do not need to build separate images for them, we will download their Docker images from DockerHub (official Docker repository). The images for MySQL ad REDIS will contain all the software and runtime to run these servers. But for our Flask server, we need to build a new image that contains not only a Python environment but the Flask-based Python code that we developed in previous stages of this tutorial, and the libraries that our code depends on.

A Docker image consists of layers, where each layer builds on top of another.
At the very bottom is the OS layer. A Dockerfile specifies the layers.
The following is the Dockerfile for our Flask app. Copy the following to a new file named "Dockerfile" in the root directory.

```dockerfile
# To build:
# docker build -t starwars-api:latest .

FROM python:3.10.3

RUN mkdir -p /app/

COPY . /app/

WORKDIR /app/

# Install GCC (C compiler) to install certain Python requirements below.
RUN apt --yes update && \
    apt --yes install gcc && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir -r requirements.txt

# No longer need GCC, uninstall it.
RUN apt --yes --purge remove gcc

RUN chmod +x run_server.sh

CMD ["./run_server.sh"]
```

The first like `FROM python:3.10.3` indicates that our Docker image will be built on top of another Docker image tagged as `python:3.10.3`. That contains not only a Linux-based OS layer but also a Python runtime (version 3.10). (Think of this a virtual environment that already has Python installed on it.) The following lines build new layers on top of it.
- `RUN mkdir -p /app/` runs a Linux command-line instruction (indicated by `RUN`). Here, we create a new directory named `app` at the root directory.
- `COPY . /app/` copies the contents of the current directory (the starwars-api) to the `/app` directory in the image.
- `WORKDIR /app/` marks `/app` as the working directory, so the subsequent commands will assume `/app` is the current directory.
- Next, we run another Linux command (it actually is multiple commands separated by `&&`, in Linux this means run the next command only after the previous comand succeeds). Here, we install the GCC compiler. This is because the following (`pip install`) command requires a C compiler.
-  `RUN pip install --no-cache-dir -r requirements.txt` runs the `pip` command to install dependencies we specified in the `requirements.txt` file. Note that, this command run on the virtual image, not in our machine. Remember that we copied all the contents of our starwars-api directory to the `/app` directory in the Docker image (with command `COPY . /app/`) so the `requirements.txt` file exists in the `/app` directory of the image. This command will install the libraries we need for our Flask app to run.
- Since we do not need a C compiler after the `pip install` command, we uninstall GCC in the next command: `RUN apt --yes --purge remove gcc`.
- Now we are ready to run our Flask app. Each Docker image has a entrypoint that is another Linux command to start the actual app the Docker image represents. The `CMD` command is one way to do it, it takes a list of tokens, the first one a program name and the others its arguments. We could run our app by saying `CMD ["python", "main.py"]`, which would correspond to running `python main.py` on the command line of the Docker image. However, we also need to initialize the MySQL database by running the `database/mysql/setup.py` script. Thus we need to run two Python scripts one after the other. How can we do this in a single command? Of course by combining them in a shell script. `run_server.sh` does exactly that.

Now, copy the following script to a new file `run_server.sh` in the starwars-api root directory:

```bash
#!/bin/bash

set -e

# Add /app to Python path for import statements find the scripts inside /app.
export PYTHONPATH=/app:$PYTHONPATH

echo "Setting up database"
python database/mysql/setup.py

echo "Running server"
python main.py
``` 

This script runs database setup script and main script in sequence.
Notice that these scripts will run from the `/app` directory in the Docker image, not in your host machine (again remember that the command `COPY . /app/` will copy this shell script to the `/app` directory when building the image).

#### Building Docker image

Now we have `Dockerfile` and `run_server.sh` scripts saved, we can build the Docker image.
For this, first, install Docker in your machine, by following the instructions at https://docs.docker.com/get-docker/

Once the Docker installation is complete, go to the starwars-api root directory in the command-line shell and run the following command:

```bash
docker build -t starwars-api:latest .
``` 

This will run the commands in the Dockerfile and build an image.
After successfully build the image, you can see the image by running `docker images` in the shell.

The argument `-t starwars-api:latest` labels the image with an *image name* (`starwars`) and *tag* (`latest`).
The tag usually indicates a version number. Here, we use `latest` to indicate it is the latest version.

#### Running Docker images

You can run the image to instantiate a container by running the following command:

```bash
docker run --name starwars-api -p 5003:5003 -d starwars-api:latest
```

This command will instantiate a container from our image (labelled `starwars-api:latest`).
The argument `-p 5003:5003` defines a port mapping from the Docker container and our host machine. It says: "whenever port 5003 is sent a message, forward it to the port 5003 of the Docker container". Thus, when we enter the address "http://localhost:5003/ui" to our browser, it forwards the HTTPS request to the Docker container's port 5003. The left-hand-side of the mapping could be any port on our host (for example, 1234), but the right-hand-side should the port our Flask app listens (5003). For example, we could instead specify this mapping as `-p 1234:5003`. In that case, we would need to test our backend with by entering address "http://localhost:1234/ui" to our browser.

#### Combining other servers with Docker-compose

In addition to our Flask app, we also need to run a MySQL server and REDIS server. They have their own images, so we do not need to build an image for them on our own. But we still need to configure these images (for example, set password and the MySQL and REDIS).
We can use the `docker run` command for MySQL and REDIS images, similarly to how we run Flask app.

But there is an easier way to do it. Docker-compose is a utility to specify the run configuration of multiple servers in a single file.
Then, we can run these servers together in a single command.
Create a new file named `docker-compose.yml` in the starwars-api root direcory and copy the following to it:

<details><summary>docker-compose.yml file</summary>

```dockerfile
# docker-compose pull
# docker-compose up -d
# docker-compose down

version: '3'

services:
  #############################################################
  api:
    image: starwars-api:latest
    hostname: starwars-api
    environment:
      STARWARS_API_HOST: 0.0.0.0
      STARWARS_API_PORT: 5003
      # This should be the hostname of the mysql service below
      STARWARS_MYSQL_HOST: starwars-mysql
      # This should be the hostname of the redis service below
      STARWARS_REDIS_HOST: starwars-redis
    expose:
      - 5003
    ports:
      - 5003:5003
    networks:
      - service-network
      - internal-network
  #############################################################
  mysql:
    image: mysql:8.0.28-debian
    hostname: starwars-mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: fathat101users
      MYSQL_ROOT_PASSWORD: 00Apassword7
    volumes:
      - mysql-data:/var/lib/mysql:rw
    networks:
      - internal-network
  #############################################################
  redis:
    image: redis:6.2.5-alpine3.14
    hostname: starwars-redis
    restart: unless-stopped
    command: redis-server --requirepass redisrocker
    volumes:
      - redis-data:/data:rw
    networks:
      - internal-network
  #############################################################
networks:
  service-network:
  internal-network:
    # The following isolates mysql and redis servers from the Internet
    internal: true
  #############################################################
volumes:
  mysql-data:
  redis-data:
```
</details>

The file specifies three services. Each service will correspond to a separate container.
Thus, we specify an image for each service. Notice the label `starwars-api:latest` for the `api` service, it is the same label we associated with our Flask app's image.
Thus, the `api` service will be served from the container instantiated from that image.

**Volumes:** Normally, when a Docker container terminates, all the new data saved while the container is running is lost. To prevent this, we mount named-volumes to directories in the Docker container. Data saved to volumes are preserved between different runs of the container, so they are not lost. In our docker-compose file, we define two volumes and mount the directories in MySQL (`/var/lib/mysql`) and REDIS (`/data`) contaainer to these volumes (volumes are actually directories in the host machine, managed by Docker).   

**Networks:** Docker creates virtual networks for the services to communicate. Here, we define two networks: `service-network` and `internal-network`. Our Flask app is connected to both networks, while MySQL and REDIS are connected only to `internal-network`. Notice the field `internal: true` in the `internal-network` definition. This makes the network be isolated form the Internet, so MySQL and REDIS servers cannot access to the Internet, and these servers cannot be accessed from the Internal, neither. Only the Flask app (because it is also part of `internal-network`) can access the MySQL and REDIS servers. This provides additional security by isolating the storage-level servers from the rest of the world.  

You can now run these three services using the following command:

```bash
docker-compose up -d
```

Notice that at the first run, Docker downloads the images for MySQL and REDIS from DockerHub, the public repository of Docker images for well-known software, as we used the official Docker images for MySQL and REDIS instead of build them on our own. You cna browse images for other well-known software at https://hub.docker.com/.
It did not download the image for our Flask app, since we built it ourselves from a Dockerfile. 

This command by default uses the `docker-compose.yml` file in the current directory.
The argument `-d` runs the services in *daemon* mode, so the command returns back to the shell and the services run on the background.
You can turn down the services using the following command:

```bash
docker-compose down
```

In summary, we have created a Docker image to contain *everything* to run our Flash app on any machine that has Docker installed.
We only need to transfer this image to the server machine and run it with Docker.
We also created a docker-compose file to specify the images and configuration for our Flask app, MySQL, and REDIS servers.
Now, if we are given a virtual machine by a cloud provider, we can just transfer our Flask app's Docker image and the docker-compose.yml file to that virtual machine and run the three servers of our backend without needing anything else.

## That's it! Congratulations you have successfully completed building your Star Wars backend API
### Give yourself a pat on the back and may the force be with you.
