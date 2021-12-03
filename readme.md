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

What happens when we do `docker container run`?
- Looks for image locally, if not, then go to Docker Hub and download the latest version
- Gives virtual IP on a private network inside docker engine
- By default, port 80 is opened on the host and forwarded to port 80 of the container 
- Starts the container by running the `CMD` command specified in the Dockerfile

## Container vs VM
- Not the same! 
- Containers are processes running on the host
- On a mac, they will appear as a process (when one does `top`) in the virtual VM that the Docker engine creates 
