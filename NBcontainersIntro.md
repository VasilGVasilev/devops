**In summary, while Docker provides a way to package and distribute applications, Kubernetes provides a way to manage and orchestrate those containers at scale in a production environment.** 

Docker is a containerization platform that allows developers to package their applications and dependencies into a single, portable container. Docker provides a consistent environment for application development, testing, and deployment across different platforms and infrastructures. Docker containers are lightweight, efficient, and can run on any operating system that supports Docker.

Kubernetes, on the other hand, is a container orchestration platform that automates the deployment, scaling, and management of containerized applications Kubernetes is designed to manage multiple containers running in a distributed environment, such as a cluster of servers or a cloud platform. Kubernetes ensures that containers are running in a healthy state, manages their resources, and provides a way to scale the application up or down based on demand.

**Class to object is like Docker Image to Docker Container**

A Docker image is like a blueprint or template for creating a Docker container, but it's not the actual container itself. When you create a container from an image, Docker uses the image as a read-only template to create a new container with a writable layer on top of it. The writable layer is created using a technology called UnionFS, which allows multiple layers to be stacked on top of each other to create a single filesystem view.

**The Docker daemon (dockerd) is the core component of the Docker engine. It is a background process that manages the creation and running of Docker containers.**

When you start a Docker container, the Docker daemon creates a container based on the image you specify and starts it up. The Docker daemon is responsible for managing the container's life cycle, including starting, stopping, and restarting containers.

The Docker daemon listens for requests from the Docker client and executes them. The Docker client communicates with the Docker daemon using the Docker API, which allows you to perform operations on containers, images, and other Docker objects.


The docker image console logging 'hello docker' contains alpine linux, node.js and app.js

    NB: Docker stores images on the machine you are using not in the directory you build it, thus, no cause for concern uploading into github repo without limit via .gitignore, but have .dockerignore file

**Docker runs only Linux distributions on its containers, linux distributions are still distributions of the same OS, VMs are the tools that can containerize nested OS**

Docker is a containerization technology that allows you to run applications and services in an isolated environment, known as a container. Docker containers do not have the ability to maintain an operating system like virtual machines do, but instead, they share the kernel of the host operating system.

This means that you can only run Linux distributions on Docker because Docker containers rely on the Linux kernel. However, it's worth noting that you can run various distributions of Linux, such as Ubuntu, Debian, CentOS, and more, within Docker containers.

**DockerHub is like a Github for containers**

**Building a container**

1. Turn on Docker
2. Have at least one file in directory (here called app.js)
3. Have a Dockerfile in directory with at the very least this code:
    ```sh
    FROM node:alpine
    COPY . /app
    WORKDIR /app
    CMD node app.js
    ```
4. docker build -t name-of-image

**alpine**
    docker pull alpine
    docker run -it alpine
    CTRL + D

**Containers are isolated instances of an image but are they?**

Volumes provide the ability to connect specific filesystem paths of the container back to the host machine. If you mount a directory in the container, changes in that directory are also seen on the host machine. If you mount that same directory across container restarts, you’d see the same files, namely, those files persist. -> *Volumes are the preferred mechanism for persisting data generated by and used by Docker containers.*

**Persisting data across containers - volume mounts as an opaque buckets of data.**

With the database being a single file, if you can persist that file on the host and make it available to the next container, it should be able to pick up where the last one left off. By creating a volume and attaching (often called “mounting”) it to the directory where you stored the data, you can persist the data. 

The volume’s contents exist outside the lifecycle of a given container. But where does Docker store that data?
The Mountpoint is the actual location of the data on the disk. Note that on most machines, you will need to have root access to access this directory from the host. But, that’s where it is.

![alt text](https://github.com/VasilGVasilev/devops/blob/main/volumes.png)

**Volumes vs bind mounts**

*Volumes are separate data entities that containers have access to.*
A volume in Docker is a way to store and manage persistent data outside of the container's file system. A volume is created and managed by Docker itself and can be shared between multiple containers. Volumes are usually the recommended way to manage persistent data in Docker because they offer better performance and security. Docker volumes can be managed using the docker volume command.

*Bind mount links data from host into container.*
On the other hand, a bind mount is a way to mount a directory on the host machine into a container. With a bind mount, a directory on the host machine is mounted into a container at a specific path. Bind mounts are managed by the user and are useful for developing and testing applications locally, as they allow changes made on the host machine to be reflected in the container immediately. Bind mounts can be created using the -v option when running the docker run command.

**Each container should do one thing and do it well**

Containers run in isolation so you need networking betweeen them for communication -> App <-> MySQL

Create network
```sh
docker network create todo-app
```

Start a MySQL container and attach it to the network
```sh
docker run -d `
     --network todo-app --network-alias mysql `
     -v todo-mysql-data:/var/lib/mysql `
     -e MYSQL_ROOT_PASSWORD=secret `
     -e MYSQL_DATABASE=todos `
     mysql:8.0
```

**Docker Compose**

Docker Compose is a tool that was developed to help define and share multi-container applications. We do that by creating a YAML file to define the services and spin everything up or tear it all down.

**Layer caching**

Images have dependencies and it is best practice to narrow down those layers of dependencies when we ship the image around.

We do that by restructuring Dockerfile so that it reflects the language specific techniques concerning dependencies

Node-based applications have their dependencies in the package.json file. Thus, best practice would be to copy package.json file first, install dependencies and only then copy everything else, meaning we only recreate the npm/yarn dependencies if there was a change to the package.json.

1. Updated Dockerfile 

```sh
 # syntax=docker/dockerfile:1
 FROM node:18-alpine
 WORKDIR /app
 COPY package.json yarn.lock ./
 RUN yarn install --production
 COPY . .
 CMD ["node", "src/index.js"]
 ```

2. Build image

```sh
docker build -t getting-started
```

3. Change something in file

4. Rebuild image

```sh
docker build -t getting-started
```

Notice that the build was MUCH faster! 


## What happens if the containers die? How do you scale across several machines? Container orchestration solves this problem. Tools like Kubernetes, Swarm, Nomad, and ECS all help solve this problem.