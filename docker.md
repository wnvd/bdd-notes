## Why is Docker needed?
Docker and other container technologies allows us to package our program 
with *an environment* and ship the whole thing. Most real world programs 
don't work in isolation, they need:

- Files from disk
- Environment variables
- Installed dependencies
- Specific permissions

Docker images and containers have become a standard for deploying applications.

We are learning Docker and containerization.


These are the things you will likely use:

- Docker server or Docker daemon (daemon is just a program that runs in the background with user interaction):
This listens to requests from the desktop app and execute them. If this isn't running nothing else will work.

- Docker Desktop:
Starting the GUI should start the server, at least that's how I usually
ensure the server is running. The GUI is the visual way to interact with
the Docker.

- The Docker CLI:
As a developer, most of your work will be interacting with Docker via
CLI.


"Docker takes away repetitive, mundane configuration tasks and is used throughout the development lifecycle for fast, easy and portable application development – desktop and cloud. Docker’s comprehensive end to end platform includes UIs, CLIs, APIs and security that are engineered to work together across the entire application delivery lifecycle."

-- The Docker team

Simply: Docker allows us to deploy our application inside "containers",
which are kind of like very lightweight machines. Instead of just shipping
an application, we can ship an application and the environment it runs in.


# Docker Hub
Docker hub is the official cloud service for storing and sharing Docker
images.
We'll use Docker Hub in this course, but there are other popular
alternatives, and they're usually coupled with cloud service providers.

- AWS ECR
- GCP Container Registry
- Azure Container Registry

# Containers
A container is a standard unit of software that packages up code and all
its dependencies so the application runs quickly and reliably from one
computing environment to another.

-- Docker

We've had virtual machines (like virtual box) for a *long* time. The trouble with
virtual machines is that they're slow (naveed: this is not entirely true because
fly.io are using virtual machines) Booting up up usually takes *longer* that a 
physical machine.

Containers on the other hand, gives us 90% of the benefits of virtual machine, but
are *super lightweight*. Containers boot up in seconds, while virtual machines can 
take minutes.

# Why are containers lightweight?
Virtual machines virtualise hardware, they emulate what a physical computer does at
lower level. Containers virtualise at the *operating system* level. Isolation between
containers that are running on the same machine is still *really good*. For the most
part, each container feels like its has its own operating system. In reality, a lot of
resources are being shared, but they're being shared securely through namespaces.

# Images
Image: A read only *definition* of a container (image is definition of application and its environment configuration)
Container: an *instance* of a virtualised read-write environment (container is simply an instance of that configuration)

A container is basically an image that's **actively running**. In other
words, you boot up a container from an image. You can create multiple separate
container all from the same image (it's kinda like the relationship between
class and object).

```bash

# To pull an image
$ docker pull <image-name>

```

# Running a container
You can run an image inside a container using 
```bash

# This is an structure example
docker run -d -p hostport:containerport namespace/name:tag

```

`-d`: Run  in detached mode (doesn't block your terminal).
`-p`: Publish a container's port to the host (forwarding).
`hostport`: The port on your local machine.
`containerport`: The port inside the container.
Here hostport:containerport is mapping the port of local machine
with the container port.


`namespace/name`: The name of the image (usually in the format `username/repo`
`tag`: The version of image (often 'latest')

To see running "Containers" use the below command
```bash

docker ps

```

After running `docker ps` command you will 'PORTS' column which will
have something like this '0.0.0.0:8965->80/tcp. This is saying that the
port 8965 on your local machine is being forwarded to port 80 on the running
container. Port 80 is conventionally used for HTTP web traffic.
You can Navigate to port 8965 on your local machine to see what container
is running (in our case web page).

# Stop a container
`docker stop`: This stops the container by issuing a `SIGTERM` signal
to the container. You'll typically want to use `docker stop`.
`docker kill`: This stops the container by issuing a `SIGKILL` signal
to the container. This is a more forceful way to stop a container and 
should be used as a last resort.


```bash

docker stop <container-id>
docker kill <container-id>

```
# Multiple Containers
It is very common to run multiple containers on a single machine.
Each container has its isolated environment and process. Even if we
start 10 containers from the same image, the image is just a definition
of the container (the code and starting environment). Each container
is a separate instance of the image.

```bash

docker run -d -p 8965:80 docker/getting-started
docker run -d -p 8966:80 docker/getting-started
docker run -d -p 8967:80 docker/getting-started
docker run -d -p 8968:80 docker/getting-started
docker run -d -p 8969:80 docker/getting-started

```
We use port 80 within each container because that's the port that
the application is binding to inside the container. It's being
forwarded to the different ports we specify the host machine.

## Volumes
By default, Docker containers don't retain any state from past containers.
For example, If I:
1 Start a container from an image
2 Make some changes to the filesystem (like installing a new package) in 
that container
3 Stop the container
4 Start a new container from the same image
5 The new container does not have the changes I made in step 2

*However, if I restart the stopped container, it will have the changes I made.
This is only with mentioning because sometimes developers think that killing
and old container and starting a new one is the same as restarting a process
but that's not true. It's more like resetting the state of the entire machine
to the original image.*

Docker does have ways to support "persistent state" through storage volumes. 
They're basically a filesystem that lives outside the container, but can be
accessed by the container.

- Creating a Volume

```bash

docker volume create ghost-vol


```

- List Volume

```bash

docker volume ls

```

- Inspect Volume

```bash
docker volume inspect ghost-vol

```

We are going to use *ghost* CMS as the example:

First we pull *ghost* image
```bash

docker pull ghost

```

Then to run the *ghost* CMS as follows:

```bash

docker run -d -e NODE_ENV=development -e url=http://localhost:3001 -p 3001:2368 -v ghost-vol:/var/lib/ghost ghost

```
Above  we did the following:
`-d` run the image in detached mode to avoid blocking the terminal.
`-e NODE_ENV=development` set an environment variable within the container.
This tells ghost to run in "development" mode.
`-e url=http://localhost:3001` set another environment variable, this one tells
Ghost that we to be able to access Ghost via a URL on our host machine.
`-p` is used for port forwarding between the container and our host machine. 
`-v ghost-vol:/var/lib/ghost` mounts the `ghost-vol` volume that we created
before the `/var/lib/ghost` path in the container. Ghost will use the 
`/var/lib/ghost` directory to persist stateful data (files) between runs.

**NOTE: `-v` isn't used for volume only it is flag for mounting, later you will
use it to mount files from local machine container server file**


## Some useful commands for Docker:

### Get running container details
```bash

$ docker ps

```

### Get all container details
```bash
$ docker ps -a
```

### Stop and remove container with rm command
```bash
$ docker stop <container-id>

$ docker rm <container-id>
```

## Persistence
A *container's* file system is *read-write*, but you delete a container and start
a new one from the same image, that new container starts from scratch again with
copy of the image. All stateful changes are **lost**.

A *volume's* file system is *read-write*, but it lives outside a single container. 
If a container uses a volume, then stateful changes can be persisted to the volume
even if the container is deleted.

## Delete a volume
you can use `docker ps -a` to see all containers, even those that aren't running.

you can use `docker rm <container-name> or <container-id>` to remove container
you can use `docker rmi <image-name>` to remove container

*NOTE*:
` -v ghost-vol:/var/lib/ghost ghost` create new volume `ghost-vol` if it didn't exist.

## Exec
You can also run commands inside docker container

```bash

# example
docker exec <container-id> <command>

docker exec d9f08734 ls # this is list the contents inside directory

```

## Live Shell
Being able to run off commands is nice, but it's often more 
convenient to start a shell session running within the container itself.

This is where `-i` and `-t` flag come it

`-i` makes the exec command interactive

`-t` give us a tty(teletypewriter) interface

tty is unix cmd to print the file naem of the terimal connected to standard input.

running `/bin/sh` give us a shell session inside the container


```bash

docker exec -it <container-id> /bin/sh

```

## Offline
Here we are forcing containers to offline mode.
Some of the reasons to remove networking ability of a container are:
- Running 3rd party code that you don't trust. 
- letting other people execute code on your machine.
- Container has virus that is sending request over internet, and
you want to inspect.

This can be achieved using `--network none` flag with `docker run ...`
command.


## Exercise
As and exercise we used caddy and used it to server index.html page

```bash

# Here '$PWD' is used to get current working directory path
docker run -d -p 8881:80 -v $PWD/index1.html:/usr/share/caddy/index.html caddy

# in exercise we did the same for index2.html and index3.html

```

## Custom Network
We can create custom bridge network so that our containers can communicate
with each other. If we want them, but still otherwise remain isolated.

We will be building a custom network where our application servers are hidden
and only our load balancer can communicated with and is exposed to the host.

- Create a Custom Bridge network
```bash

docker network create caddytest

```

- Listing all the networks
```bash

docker network ls

```

In exercise we stopped and restart the containers and attached them 
with the network crated

**Note: using 'restart', 'start' command did not work I had to delete
the old container and spin up new ones**

**Reminder: I didn't used '-p' flag as we are not binding container to 
specific ports and only accessing them using network we created (caddytest).**

Notice we provided name for the container using '--name' flag and 
used '--network' flag to attach it to the previously created network.

```bash

$ docker run -d --network caddytest --name caddy2 -v $PWD/index2.html:/usr/share/caddy/index.html caddy

```


We then create a new container and attached it to the same network 
that previous containers were attached to and were only able to access those containers
using this new container

```bash

$ docker run -it --network caddytest docker/getting-started:latest /bin/sh

$ curl caddy1

$ curl caddy2

```

## Configuring the Load Balancer
Now we will be configuring the Load Balancer (caddy) which is on the 
same network

First, we stop the container servers 'caddy1' and 'caddy2'.

Then we crate a new file in our local directory called 'Caddyfile'

This tells Caddy to run on `localhost:80` and to round robin any 
incoming traffic to `caddy1:80` and `caddy2:80`. This on works when we 
are using load balancer on the *same network*,

```Caddyfile

localhost:80

reverse_proxy caddy1:80 caddy2:80 {
	lb_policy	round_robin
}

```

Lastly, we start the load balancer on port '8880', and this time we 
provide the 'Caddyfile' to mount.

```bash

$ docker run -d --network caddytest -p 8880:80 -v $PWD/Caddyfile:/etc/caddy/Caddyfile caddy

```
Remember to start the stopped containers 'caddy1' and 'caddy2'.

## Dockerfile
Till now we have been using docker to run other people's software, but it 
can also be used to build and package our own software.

Docker images are build from *Dockerfiles*. A Dockerfile is just a text
file that contains all the commands needed to assemble an image. It's essentially
the **Infrastructure as Code**(IaC) for an image. It runs commands from top
to bottom, kind of like a shell script.

Instead of manually installing dependencies on servers and making updates manually,
We can check a Dockerfile into source code and build it automatically *automation*.


- Create an image from a 'Dockerfile'
```bash

filename: Dockerfile
# This is a comment

# Use a lightweight debian os
# as the base image
FROM debian:stable-slim

# execute the 'echo "hello world"'
# command when the container runs
CMD ["echo", "hello world"]

```

- To build the image from Dockerfile
```bash

# -t flag tags the image name with "helloworld" and the "latest" tag
# Note: names are used to organize your images and tags are used to keep
# track of different versions

docker build . -t helloworld:latest

```

- Run image built from Dockerfile

```bash

docker run helloworld

```

If you use `docker ps` to see the running containers you will find *not*
"helloworld" container that we built using image. All it did was print
and exit.

Just like regular programs, docker containers can execute simple commands
that exit quickly, or they can execute servers that run until killed. It
just depends on the command you give it.

You can see 'helloworld' container in stopped container using 
`docker ps -a`


## Dockerizing a Server

Following are the steps to dockerize a Server:
1 - Build a Server
2 - Create a Dockerfile
3 - Build an image using the Dockerfile
4 - Run the image in a container

### Creating a Dockerfile
```Dockerfile

# Starting with a simple lightweight *Debian Linux OS*
FROM debian:stalbe-slim


# COPY source to destination
# here 'goServer is the the name of our server
# We can also use 'ADD' command here but 'COPY'
# is fine we don't need extra functionality that 'ADD' offers
COPY goserver /bin/goserver 

# add 'CMD' command as the last line in the 'Dockerfile'
CMD ["/bin/goServer"]

# Build you Dockerfile into an image
# -t flag stand for tag
# tag name can only be lowercase
docker build . -t goserver:latest



# Build a new container from the image
docker run -p 8010:8010 goServer


```

[NOTE]
If you get an exec format error, it's probably because you
built the go server for your local architecture, but you're
trying to run it on a Linux OS! To fix it, rebuild the binary
(and then the Dockerfile) with these flags:

`GOOS=linux GOARCH=amd64 go build`

You should be able to access your server on port you are running it on.

You can also define environment variables inside the `Dockerfile`

```Dockerfile

# This env var will be defined inside the container
# make sure to define with before running 'CMD'.
ENV PORT=8999

```


## Dockerizing programs that are not Go 
We contained go app till, we didn't require shipping dependencies and a 
complier/interpreter will our app because it is the beauty of go.

The following app are the containerized the same way:
- Python
- JS/TS
- Java
- Ruby
- PHP
- C#
- etc...

They all have runtime dependencies.

In a language like python this would go something like this.

In course we named our file as `Dockerfile.py`
and passed `-f Dockerfile.py`
```Dockerfile

# this is our base
FROM debian:stable-slim

# copying our files
COPY books/ books/
COPY main.py main.py

# the command to run the program
CMD ["python3", "main.py"]


```

Now we build the image from `Dockerfile.py`
`docker build -t bookbot -f Dockerfile.py .` 

But when we run the image, we will get an error
telling use that it contained environment does not
know what 'python3' is, meaning the contained environment
does not have python interpretor installed.

`docker run bookbot`

To fix this we are going to use 'RUN' command

`RUN install apt-get python3`

```Dockerfile


# this is our base
FROM debian:stable-slim

# copying our files
COPY books/ books/
COPY main.py main.py

# install dependencies
# you have to run `apt-get update` before install
# otherwise you get an error
RUN apt-get update && apt-get install -y python3

# the command to run the program
CMD ["python3", "main.py"]


```

Here you need to know difference between RUN and CMD commands:

- **RUN** builds the image by executing commands.
- **CMD** defines what command runs when the container starts.


## Docker Logs
When containers are running in detached mode using '-d' flag, you 
don't see any logging, but what if something goes wrong This is were

`docker logs [options] <container-name-or-conainer-id>` comes in.

You can also use '-f' flag to follow logs in real time
`docker logs -f <container-name/id>` this will follow logs happening
in the container in real time.

You can also use '--tail <number>' flag to just print last 'number'
of logs in the container.

```bash
# example 

# this will spin up a alpine container and execute the below script
# repeatedly
docker run -d --name logdate alpine sh -c 'while true; do echo "LOGGING: $(date)"; sleep 1; done'

# here we use log command to check the logs
docker log -f logdate

docker log --tail 5 logdate

```

## Stats
We learned how to inspect a container logs, we can also see the resource
utilization.

Pretty common to spin up a docker container and forget about them and 
wonder why host machine has got slow. It's really nice to see how much
RAM/CPU each container is using and it's *critical* in production environment.

The `docker stats` commands give you a live stream of resource usage
for running containers.

`docker stats [option] [container]`

Now we are going to spin up some containers to use the command


Below, are handing container 2 CPUs for utilization
```bash

docker run -d --name cpu-stress alexeiled/stress-ng --cpu 2 --timeout 10m

```


Below, are handing container 1G of ram for utilization (which is a lot 
for a single container)
```bash

docker run -d --name mem-stress alexeiled/stress-ng --vm 1 --vm-byes 1G --timeout 10m

```

## Top

The `docker top` command show the running *processes* **inside** a container.

```bash

docker top container [ps options]

```
Use `stats` for *entire* container and `top` for *processes* **inside** container.

## Resource Limits
When you notice a container is using too many resources, if you don't have the time or the ability to "fix" the code, you can limit the resources the container has available. The `docker run` command has a few options for
limiting resources:

- `--memeory`: Limit the memory available to the container
- `--cpus`: Limit the CPU shares available to the container

## Publishing to Docker Hub
Docker hub is the official cloud service for storing and sharing Docker images. We call these kinds of services "registries".
There are other popular image registries include:
- AWS ECR
- GCP Container Registry
- Github Container Registry
- Harbor
- Azure ACR

Docker organizes images in repositories that container different tags/versions of the same image name.


## Publishing to Docker hub
You can publish you image after building it using `docker push <image-name>`

Example:
`docker push wnvd/goserver`

You can also pull image:
`docker pull wnvd/goserver`

## Tags

Tag is a label that you can assign to a specific version of an image,
similar to a tag in Git.

### Deployment Pipelines
Publishing new version of Docker images is a very common method of 
deploying cloud-native back-end servers. 

Below is the hierarchy describing a pipeline of many production-systems

1 - Code is merged to the main branch.

2 - Code is built into an executable.

3 - Docker image is built with all dependencies.

4 - New docker image version is pushed up to a registry.

5 - Production sever pull down latest version.

## Latest (tagging)
This is something very important regarding semantic versioning/tagging.

As we know by default image is tagged as latest.

What the workflow should look like is something like this:

When we build an image we tag it twice, one with *semantic versioning*
e.g '0.2.0' and other with *latest* tag.

So when we push the new version of the image we use *semantic versioning*
and then point the *latest* tag to the newer version, this way the older
image version still has its own *semantic versioning* but loses the *latest*
tag as it is no more the newest version.

## The deployment Process
- The developer (you) writes some new code
- The developer commits the code to Git
- The developer pushes a new branch to GitHub
- The developer opens a pull request to the main branch
- A teammate reviews the PR and approves it (if it looks good)
- The developer merges the pull request
- Upon merging, an automated script, perhaps a Github action, is started
- The script builds the code (if it's a compiled language)
- The script builds a new docker image with the latest program
- The script pushes the new image to Docker Hub
- The server that runs the containers, perhaps a Kubernetes cluster, is told there is a new version
- The k8s cluster pulls down the latest image
- The k8s cluster shuts down old containers as it spins up new containers of the latest image


