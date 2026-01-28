# Containers with Docker

## What is a container?

Container is a running, lightweight package of an application plus everything it needs to run—code, runtime, system tools, libraries, and settings—but without bundling a whole operating system - it uses host machine kernel.

Docker is the most popular container technology but not the only one - there's also Containerd, Cri-O etc.

## Container vs Image

Container is a running instance of a blueprint which is Docker Image. Docker containers are layered by design. Layers are often shared between versions of the image - that makes fetching and running newer version of the container much quicker than the first one.

To run Docker container simply use:

`$ docker run image:tag`

## What is a difference between container and an vm?

Docker by design virtualize OS application layer whereas VM consists of both OS application layer as well as the kernel. This conceptual difference brings the following:

- Docker images are much smaller than then VM images
- Docker launches much faster than the VM
- VM is compatible with all of the OS where Docker is native for Linux only.

Compatibility issue on Windows and Macs is solved by running a small Linux VM in which Docker Desktop runs docker.

## Docker Architecture and its components

Docker Engine has three main components:

1. Server - pulls images and manages them - it forks into subsystems of container runtime, volumes and network as well as builder

2. API - interacting with Docker Server

3. CLI - client to execute Docker commands

## Main Docker Commands

`$ docker pull image:tag` - used to pull an image from the registry i.e. DockerHub

If run with -d it detaches.

`$ docker run image:tag` - same as the above but pulls the image and then runs the container

`$ docker images ls` - used to display downloaded images

`$ docker start/stop <container_id>` - used to start/stop container

`$ docker ps -a` - used to list running and not running containers

Container port is not the same thing as host port - one needs to be bind to the port of another (container -> host). Otherwise the container won't be accessible.

`$ docker run -p host_port:container_port image`

If run with --name you can specify custom name of the container.

## Debug Docker Commands

`$ docker logs container` - used to display container's logs

`$ docker exec -it container /bin/bash` - used to access internal shell of the container

## Developing with docker

1. Pull mongo and mongo-express from DockerHub:

`$ docker pull mongo && docker pull mongo-express`

2. Create isolated docker network for the app - so that apps can talk to each other using container names:

`$ docker network create mongo-network`

3. Run containers and specify in which network they should be in:

```
$ docker run -d -p 2717:2717 \
-e MONGO_INITDB_ROOT_USERNAME=admin \
-e MONGO_INITDB_ROOT_PASSWORD=password \
--name mongodb \
--net mongo-network \
mongo
```

```
docker run -d -p 8081:8081 \
-e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
-e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
-e ME_CONFIG_BASICAUTH_USERNAME=user \
-e ME_CONFIG_BASICAUTH_PASSWORD=pass \
-e ME_CONFIG_MONGODB_SERVER=mongodb \
-e ME_CONFIG_MONGODB_URL=mongodb://mongodb:27017 \
--net mongo-network \
--name mongo-express \
mongo-express
```

4. Make changes to the NodeJS application so that it can use MongoDB that was just started with Docker -

5. Build the application and run it:

`$ cd app`
`$ npm install`
`$ node server.js`

Docker comes with a tool that lets you define and run multi-container Docker applications using a single config file (usually docker-compose.yml) - **docker compose**. In docker-compose.yml file you define services in YAML format.

When using docker compose it automatically creates a network named after partent folder of a project compose file is placed in and connects containers to it - of course unless you tell it otherwise.

By default data is not persistent in containers.

To run your compose file:

`$ docker compose -f docker-compose.yaml up -d`

To stop containers from running:

`$ docker compose -f docker-compose.yaml down`

## Dockerfile - Build your own Docker Image

Dockerfile is a blueprint for creating docker images.

Docker file keywords:

FROM - defines base image that your custom image is going to be built on

ENV - sets environment variables inside the image

RUN - executes commands at build time (inside of the container) and commits the result to the image

COPY - copies files from your build context into the image

WORKDIR - sets the working directory for subsequent instructions

CMD - specifies the default command to run when the container starts

To build an Image you run:

`$ docker buildx build --platform linux/arm64,linux/amd64 -t my-js-app:1.0 .`

In my case I'm using **buildx** builder instead of the "classic" one and build multi-architecture image since whole thing is built on Apple Silicon machine.

To run the image:

`$ docker run my-js-app:1.0`

## Private Docker Repository

Nexus set up earlier makes a perfect choice for the private repository for Docker. New repository and user role needs to be created using the web interface. Creating **connector** for docker client is necessary as well.

You need to activate Docker Bearer Token Realm in Nexus.

First step of pushing docker artifact to the repository is to authenticate:

`$ docker login repository_url:port`

Image naming in Docker registries defaults to:

`registryDomain/imageName:tag`

To push an image to remote repository first it needs to be tagged with the repository address and port:

`$ docker tag my-js-app 192.168.0.78:8083/my-app:1.0`

After that image can be pushed to the remote registry:

`$ docker push 192.168.0.78:8083/my-js-app:1.0`

## Deploy docker image on a server

To deploy the image on the server you can use docker-compose.yml file in which an image is defined as follows:

`image: 192.168.0.78:8083/my-js-app:1.2`

## Docker Volumes - Persisting Data

A Docker volume is a Docker-managed storage mechanism used to persist data outside a container’s writable layer. Volumes live on the host (or a remote storage backend), but Docker controls their lifecycle and location, not your container image.

There are several types of volumes:

- bind mounts
- anonymous
- named volume

To create persistent docker volume:

`$ docker volume create --name nexus-data`

After that we can run Nexus as Docker container:

`$ docker run -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3`

## Docker Best Practices

1. Use official docker images as base image
2. Use fixed image versions - not latest
3. Use small official images unless full blown OS is absolutely necessary
4. Optimize caching image layers - if one layer changes all the next ones are rebuilt. Use commands from least to most frequently changing. Tip: docker history - shows layers
5. Use .dockerignore file
6. Exclude build dependencies from the images using multi-stage-builds
7. Don't use root to run the app in the container if not necessary
8. Scan your images for vulnerabilities (docker scout cves image:tag)
