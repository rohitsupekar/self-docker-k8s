Following are my notes taken during this course on [Docker and Kubernetes](https://www.udemy.com/course/docker-mastery/). 

# Why Docker?

- Develop, build, test, deploy faster. 
- Allows packaging applications regardless of infrastructure, dependencies, OS, etc. 
- Maintenance is easier 
- Containers do not need re-writing code, so current applicaitons can be moved to containers 
- Container orchestration allows for reduction in VM costs 
- Open source 

# Installation 
- Two editions of Docker: CE (community edition; free) and EE (enterprise edition)
- Versions: "edge" is the beta version that comes out monthly; "stable" is well, stable!
- Mac doesn't natively support Docker (as yet), Docker app starts a VM in the background 
- There are docker versions specific to AWS/Azure/Google
- Download the mac version from the website, don't use brew :) 

# Creating containers 
- Run command `docker version` to make sure the client can talk to the docker engine 
- Image: application we want to run 
- Container is an instance of the image running as a process 
- Docker Hub is Docker's default image "registry"
- Run an nginx server in a container: 
```bash
docker container run --publish 80:80 --detach nginx
```
`--detach` runs the container in the background. 

Some useful commands 
- `docker container ls` : lists all running containers 
- `docker container ls -a`: lists all containers 
- `docker container logs <container name>`
- `docker top <container name>` : shows processes inside the container 
- `docker ps` : shows both running and stopped containers (old command)
- `docker image ls`: show all the local images available

What happens when we do `docker container run`?
- Looks for image locally, if not, then go to Docker Hub and download the latest version
- Gives virtual IP on a private network inside docker engine
- By default, port 80 is opened on the host and forwarded to port 80 of the container 
- Starts the container by running the `CMD` command specified in the Dockerfile

## Container vs VM
- Not the same! 
- Containers are processes running on the host
- On a mac, they will appear as a process (when one does `top`) in the virtual VM that the Docker engine creates 
- Docker containers are no longer just linux based. Now it is possible to create Windows containers as well. 

## What's going on inside a container

- `docker container top <container>` : lists process inside a container
- `docker container inspect <container>` : gives a json with all the data of how the container was started
- `docker container stats`: a live view of resources used by containers

### Getting a shell inside

- no SSH needed!
- `docker container run -it`: start a new container interactively
- For example: `docker container run -it --name nginx nginx bash` [this is starting a container with name ngix and image nginx and the first command run in the container is `bash`; hence we get an interactive terminal inside the container]
- `docker container exec -it` : run addtional command in existing container
- `alpine` is a linux image that's only about 5 MB in size, so its very light, it doesn't even have `bash`. 

## Docker Networks

### Concepts

- each container is connected to a private virtual network "bridge", which is attached to the host ethernet
- defaults for networking just work out of the box, but they are configurable
- `docker container run -p XY:AB` -p stands for publish.XY is the host port and AB is the container port.
- `docker container port <container>` to find the ports exposed by the container

### CLI Management

- `docker network ls` : lists all the networks 
- `docker network inspect` : inspect all the details of the networks 
- `docker network create --driver` : create a network
- `docker network connect` : attach a network to a container
- `docker network disconnect` : detach a network from a container
- the default network is called `bridge` which is connected to the physical network through the host firewall
- `docker container run --network <network_name> <image_name>` connects the new container to the network you specify

### Security
- since containers can all be connected to the same virtual network, they can talk to each other behind the host firewall without going through the physical network

### DNS
- can't use IP addresses to talk to containers since they are variable and not static
- Docker has a built-in DNS server that containers use by default
- Create new networks to host all the apps so they can talk to each other through it
- Use the `--net-alias` flag to give a DNS name to your container

**Note**: if we need to run a container and check if something it quickly and ask docker to remove everything about it once we exit, use the `--rm` flag with `docker container run`. 

## Images

- Binaries and dependencies exist in an image, it's not a complete OS
- They can be as small as a file or as big as an ubutu system

### Docker hub

- only repository of images including official ones as well as user created
- images are tagged, one version can have multiple tags
- downloading an image: `docker pull <image-name:version-tag>`
- best practice: specify the exact version upto the minor/patch version release
- `alpine` images are lightweight linux images as compared to their debian counterparts 

### Image layers

- Images are designed using the "union file system", they comprise of multiple layers
- `docker history <image-name>` can list all the different layers of the image
- Example: we start with a base ubuntu image, then through apt-get command in the Dockerfile, we install something. That's another layer. Then we set Env variables, that's another layer. So this overall image comprises of 3 layers. 
- Layer data is stored on the file system, and need not be re-downloaded. That is, two images with some common layers share a single copy of those layers. 
- `docker inspect <image-name>` gives metadata of the image in the form of a json

### Image tags

- Tag: isn't much of a version or a branch. We can think of it as a tag on github repos. 
- The same image can have multiple tags
- Tagging an image: `docker image tag <old-image:tab> <new-image:tag>` (the tag is optional)
- To push an image to docker hub, `docker image push` [login to docker using `docker login` before that]

### Dockerfile basics

- Recipe for creating an image
- Different commands in the Dockerfile are called "stanzas"
- `FROM` command to specific base image
- `ENV` to inject environment variables
- Each command in Dockerfile creates a new layer. So multiple bash commands are typically written together in one `RUN` line. For example, one can use `&&` to group different bash commands
- `EXPOSE`: ports to expose 
- `CMD`: required command, it is the command that the container running this image will execute on its entry

### Building images
- `docker image build -t <tag> .` in the folder in which you have the Dockerfile
- use the flag `-f` to specify a Dockerfile with a specific name

### Extending official images
- `WORKDIR` to change working directory, avoid using `RUN cd /path/to/dir`
- `COPY` to copy files from build source to the image
- Required commands such as `EXPOSE` and `CMD` need not be in the Dockerfile if the base image has those commands

### Pruning
- `docker image prune` cleans up "dangling images"
- `docker system prune` cleans up everything
- `docker image prune -a` will remove all images you aren't using
- `docker system df` to see space usage

## Container lifetime and persistent data

- Containers are usually immutable and ephemeral
- "immutable infrastructure": only re-deploy containers, never change
- "Separation of concerns": unique data can be recycled/reused while re-deploy the application binaries in the container
- This unique data is called "persistent data". Two ways:
  - Volumes: make a special location outside the container UFS. Volumes need a special cleanup, just deleting the container is not enough!
  - Bind mounts: link container volume path to host path 

### Persistent Data: Volumes

- `VOLUME`: command to create a new volume. If this is used
- `docker volume ls` lists all the volumnes
- `docker volume inspect <volumne-name>` gives metadata to find where these data are on the host
- For usefriendly behavior, use named volumes. Use this as `docker container run -d --name <container-name> -v <volume-name>:<volume-path> <image-name>` [reminder: `-d` is `-detached` for running container in the background]
- When creating a new container, if an already existing volume path is used, docker won't create a new volume but will reuse the already existing one
- `docker volume create` to create a volume ahead of time outside the Dockerfile

### Persistent Data: Bind Mounting

- Maps host files or directory to those of the container
- Can't use in Dockerfile, must be used with `docker container run`
- `docker container run -v <host-path>:<container-path>` to specify the bind mount. The `host-path` can be something like `/Users/asdf/path/to/dir`.
- This is great for local development since one can change files on the host, while the container is running. It will be able to see and read in those changes. One does not need to go into the bash shell of the container. 

## Docker Compose

- Why? 
  - Configure relationships between different containers
  - Save docker container run settings in readable file
  - Create one-liner developer startups

- What is needed? 
  - yaml file: `docker-compose.yml` 
    - `docker-compose` CLI tool

### `docker-compose.yml`

- version is specified at the top of the file
- `sevices`: containers
  - specify `<servicename>`: DNS name of a container
  - `depends-on`: what other containers does this container depend on?


  ### `docker-compose` CLI

  - Idea for local development, is not production-grade
  - Common commands:
    - `docker-compose up`: setup volumes/networks and start all containers
    - `docker-compose down`: stop all containers and remove containers/volumes/networks
    - `docker-compose` has the same options as `docker` available like `-d` (background), `logs`, etc.
    - `docker-compose down -v` removes volumes while shutting containers

  ### Using compose to build

  - Compose can build images and store them in cache, it won't build the images if they are found in the cache
  - Specify building an image through the `build` key in `docker-compose.yml`
  - 