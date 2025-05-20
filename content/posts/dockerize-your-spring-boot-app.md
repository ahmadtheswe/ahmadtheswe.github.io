+++
date = '2023-06-02T23:20:11+07:00'
draft = false
title = 'Dockerize Your Spring Boot App'
author = 'ahmad'
categories = ['docker', 'java']
tags = ['docker', 'spring-boot', 'java']
featuredImage = '/images/dockerize-your-spring-boot-app/featured.png'
+++
Docker has revolutionized the way how we develop our apps. It removes the "it works in my machine" problem from the stack of problems during development software.

In my opinion, Docker is one of the mandatory things you should add to your tech stack as many job applications (mostly for back-end engineers) add knowledge in Docker as their requirement. So, learn Docker to stay up-to-date!

## What is Docker

Let's see how popular platforms describe what Docker is.

Docker is a software platform that allows you to build, test, and deploy applications quickly. Docker packages software into standardized units called [containers](https://aws.amazon.com/containers/) that have everything the software needs to run including libraries, system tools, code, and runtime. Using Docker, you can quickly deploy and scale applications into any environment and know your code will run. -- [**AWS**](https://aws.amazon.com/docker/) \--

Docker is an open-source platform that enables developers to build, deploy, run, update and manage *containers*—standardized, executable components that combine application source code with the operating system (OS) libraries and dependencies required to run that code in any environment. -- [**IBM**](https://www.ibm.com/topics/docker) --

Docker is an open platform for developing, shipping and running applications. With Docker, you can separate your applications from your infrastructure and treat your infrastructure like a managed application. Docker helps you ship code faster, test faster, deploy faster, and shorten the cycle between writing code and running code. Docker does this by combining kernel containerization features with workflows and tooling that helps you manage and deploy your applications. -- [**Google**](https://www.cloudskillsboost.google/focuses/1029?parent=catalog) --

In short, Docker can be a tool that can make your development journey run smoother and faster than without it. It allows you to build, package, and deploy applications quickly. It helps you to manage dependencies to run your app. Docker also facilitates faster code shipping, testing, and deployment, reducing the time between code development and execution.

There are 6 core components of Docker you must know :

1. **Docker Engine**. The core component of Docker. It is responsible for building and running containers. Docker Engine manages the lifecycle of containers, including image management, container execution, and resource isolation. It consists of :
    - **the Docker daemon**, which runs on the host machine.
    - **the Docker client**, which provides a comment-line interface (CLI) for interacting with the daemon.

2. **Docker Images**. A read-only template that contains the necessary files and dependencies to run an application in a container. Images are built from `Dockerfile`, which are text files that specify the instructions for creating the image. Images also can be pulled from [Docker Hub](https://hub.docker.com/) or other image registries.

3. **Docker Containers**. A running instance of Docker Images. It provides an isolated environment application for the application to run, ensuring that the application and its dependencies are consistent across different environments.

4. **Docker Registry**. A central repository for storing and distributing Docker Images such as [Docker Hub](https://hub.docker.com/).

5. **Docker Compose**. A tool for defining and managing multi-container applications. It uses a YAML file to describe the services, networks, and configurations required for an application.

6. **Docker Swarm**. A native clustering and orchestration solution provided by Docker. It allows users to create and manage a swarm of Docker nodes, forming distributed system.


## How Docker Works

To know how Docker works, it's better to compare it with the older technology, a virtual machine (VM).


![Container vs VM](/images/dockerize-your-spring-boot-app/dockerized-1.webp)

I took this image from [AWS - What is Docker?](https://aws.amazon.com/docker/) article.

In a traditional VM, each application stands above an independent virtual OS. Of course, this approach provides perfect isolation for each application. But, this gives us a huge cost to run all VMs as a consequence. That consequence is the problem we want to solve by using Docker. Instead of virtualizing an OS for each application, we can leverage Docker Engine to manage all applications (as containers) to run independently (fully isolated). It's also more simple to manage dependencies with Docker instead of VM.

## Let's Dockerize your Spring Boot App

For this project, you can use my provided repository [here from my GitHub](https://github.com/justahmed99/spring-boot-docker-tutorial) as an example.

The first step, you need to build your Spring Boot app. By using this command below. It will produce a runnable `.jar` file from your Spring Boot app.

```bash
$ ./gradlew build
```

Secondly, make a `Dockerfile` file inside your project's root directory. Fill the file with the code below.

```dockerfile
FROM openjdk:17-oracle
EXPOSE 8080
RUN mkdir -p /app/
ADD build/libs/spring-docker-demo-0.0.1-SNAPSHOT.jar /app/example.jar
ENTRYPOINT ["java", "-jar", "/app/example.jar"]
```

Thirdly, make a `docker-compose.yml` file in your root directory with the code below.

```yaml
version: '3'
services:
  example-project:
    ports:
      - 8080:8080
    image: example-project
    build:
      context: .
      dockerfile: Dockerfile
    container_name: example-project
    restart: always
```

Fourthly, we use the `docker-compose` command to build a docker image, create the container from the image, and then run it. It will take a while sometimes because we need to download the necessary dependencies to run the container.

```bash
$ docker-compose up -d
```

Sixthly, after installation is finished. You can check whether it is successful by using `docker ps` command. If you can find your new container on the list, you have succeeded and it is already running.

```bash
$ docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                    NAMES
133109f2d973   example-project   "java -jar /app/exam…"   20 seconds ago   Up 18 seconds   0.0.0.0:8080->8080/tcp   example-project
```

Lastly, this means you already installed your Spring Boot app in a docker environment. You can try to hit the API using HTTP client app link curl, Postman, or even directly from your browser.

```bash
$ curl http://localhost:8080
{"message":"Hello World!"}
```

## Conclusion

From now, let's learn Docker! It is necessary to add to your skill set for becoming an up-to-date software engineer.

Wish you have a good experiment moment!