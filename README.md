# docker-demo-my-app
* A small node js + mongo db demo app to learn to develop with docker containers

## Notes on Docker

* Youtube Channel: TechWorld with Nana
	Docker Tutorial for Beginners [FULL COURSE in 3 Hours]
	https://www.youtube.com/watch?v=3c-iBn73dDE
* Demo code of the above YouTube tutorial: https://gitlab.com/nanuchi/techworld-js-docker-demo-app/-/tree/master
* My code repo: https://github.com/amitshokeen/docker-demo-my-app


**```$ docker pull  <image name>```** 
> pulls the image from the repository to the local environment

**```$ docker run <image name>```** 
> basically combines docker pull and docker start

**```$ docker stop <container ID>```** 
> stops the docker container. In place of ID, name can also be used

**```$ docker start <container ID>```**
> starts the docker container. In place of ID, name can also be used


**```$ docker run -d -p6000:6379 redis```**
> -d stands for detached mode
> -p allows to bind port of the host to the container port. Here port 6000 of this computer is bound to port 6379 of the container redis

**```$ docker ps -a```** 
> used to show the running containers if no option is used.
> The -a option, if used, will show all the containers whether running or not.

**```$ docker images```**
> gives all the images you have locally

**```$ docker logs <container ID>```**
> to help debug
 - now these logs could be very long… so you can just see the last part using the below command:

**```$ docker logs <container ID> | tail```**
> or u could just stream the logs using the below command:

**```$ docker logs <container ID> -f```**

**```$ docker run -d -p6000:6379 --name redis-older redis:4.0```** 
> --name will help to give a name of your choice

**```$ docker exec -it <container ID> /bin/bash```**
> this will take you inside the container and the next prompt will be from inside the container. You can use this for debugging etc.’
If /bin/bash does not work, then try /bin/sh

            *~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*


### Docker Network 
* Isolated Docker Network
* Two containers can talk to each other using just the container name
* The app that runs on the host can connect to them using something like:
```localhost:<portnum>```

```$ docker network ls```
> lists the docker networks

```$ docker network create mongo-network```
> this will create a docker n/w

### start mongodb
```
$ docker run -d \
   -p 27017:27017 \
   -e MONGO_INITDB_ROOT_USERNAME=admin \
   -e MONGO_INITDB_PASSWORD=password \
   —net mongo-network \
   —name mongodb \
   Mongo
```
### start mongo-express
```
$ docker run -d \
   -p 8081:8081
   -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
   -e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
   -e ME_CONFIG_MONGODB_SERVER=mongodb \
   --net mongo-network \
   --name mongo-express \
   mongo-express
```
> here **-e** is for the environment variable and **--net** is for the network name

                    *~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*


### Docker Compose
Docker Compose takes care of creating a common network. So the equivalent of ```--net mongo-network``` will not be needed in the docker compose file that has the services of mongodb and mongo-express listed.

A sample yaml file: mongo.yaml
____________________________________________________________
```
version: '3'
services:
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
  mongo-express:
    image: mongo-express
    ports:
      - 8080:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
```
____________________________________________________________
> be careful about indentations in the yaml file

 Now run the below command to finally **up** the docker services mentioned in the yaml file
> **```$ docker-compose -f mongo.yaml up```**

* Imp point: if the container, for instance mongodb, is restarted, then its data will be lost. This is undesirable.
* Therefore to enable persistence of data, **Docker Volumes** should be used.

If you want to ‘down’ the docker services, then:
> **```$ docker-compose -f mongo.yaml down```**

                    *~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*


### Dockerfile
Used to create your own image. It is basically the automation of docker image creation.

> https://www.youtube.com/watch?v=LQjaJINkQXY

Dockerfile must be named **Dockerfile**

Sample Dockerfile:
____________________________________________________________
```
# From the base image. https://hub.docker.com/_/node has supported tags listed
FROM node:15.5.0-alpine3.10

# These env variables are best kept in the yaml file. Here it is just for sample
ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PWD=password

# This command will run inside the docker image
RUN mkdir -p /home/app

# This will run on the host machine. It will copy the contents of current directory (.) to /home/app
COPY . /home/app

# This is the entry level command that will run inside the image being created
CMD ["node", "/home/app/server.js"] 
```
____________________________________________________________

* Now, to create/build the image:
> **```$ docker build -t docker-demo-my-app:1.0 .```**
>> **-t** is for the tag and name of the image
   [ **.** ] this dot stands for the location of the Dockerfile. Here it stands for the current directory

* Whenever you **re-adjust** the **Dockerfile**, you must **rebuild** the image again.
For that you must remove the image with this command:
> **```$ docker rmi <image id>```**
> After running this command, you may run into an error like:
**```Error response from daemon… image is being used by stopped container…```**

* so you got to delete the container first. To do that:
> **```$ docker ps -a | grep docker-demo-my-app```**
> This will provide the required docker ID


* Now run this command:
> **```$ docker rm <docker ID>```**


* So once the container has been removed, you can go ahead with removing the image
> **```$ docker rmi <image id>```**
    

* Now you may rebuild the docker file using 
> **```$ docker build -t …```**


* & then docker run the image again

                    *~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*

### Docker private repository (also called as a registry for Docker)

* on **AWS** you can use the **ECR** service (Elastic Container Registry)
* one can also use **Digital Ocean** or something else for the docker private registry.
>This hour long tutorial by Travis Media has an example of using the Digital Ocean: https://www.youtube.com/watch?v=i7ABlHngi1Q&t=2s

* In AWS first use the ECR service and create a repository
> In AWS you will have to then login to the docker repo... & for that AWS shows you the way **[ View push commands ]**.
You will need the **AWS CLI** and its credentials configured.
(Or watch this https://www.youtube.com/watch?v=3c-iBn73dDE from 2:04:36 onwards).
> Now do the docker push to the repo on AWS.
	* But if you simply do **```$ docker push my-app:1.0```** -> this will not work because you are not telling docker the domain.
	* So you must first tag the image (basically rename the image to include the domain in the name as well).
	* This info about the tagging is given by AWS in Push commands for my-app.
	* e.g. **```$ docker tag my-app:latest 664574038682.dkr.ecr.eu-central-1.amazonaws.com/my-app:latest```**
	* Now go ahead with the docker push using the new name (this is also given in AWS)
	
____________________________________________________________________
#### Image naming concepts in Docker repositories

* **```registryDomain/imageName:tag```**

> Its quite long when using docker repos, but with docker hub it was simple as the shorthand could be used with docker hub.
* In DockerHub it is like this:
	**```$ docker pull mongo:4.2```**
* That’s a shorthand that docker hub allows. 
		* Actually it should be: 
        >**```$ docker pull docker.io/library/mongo:4.2```**
	
* in AWS ECR: 
> **```$ docker pull 520697001743.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.0```**	
____________________________________________________________________

                    *~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*


### Deploying the application to the server

* so now that the image has been pushed to the private docker repo on AWS, we must now add to the **mongo.yaml** file as indicated below:
____________________________________________________________
```
version: '3'
services:
  my-app:
    image: 664574038682.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.0	
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
  mongo-express:
    image: mongo-express
    ports:
      - 8080:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
```
____________________________________________________________
> This Docker-Compose file would be used on the server to deploy all the applications/services

> Now use the AWS ECR hints to help login to the docker repo from the server where deployment has to be done.
> That login to the private docker repo from the server would look like this:

> **```$ $(aws ecr get-login —no-include-email —region eu-central-1)```**

* After this create the **mongo.yaml** file, as given above, in this server. Using the **vim editor** would be good.
 * After this start all the three containers ( my-app, mongodb, mongo-express ) using the docker-compose command on the server

> **```$ docker-compose -f mongo.yaml up```**

                    *~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*


### Docker Volumes
*  needed for Data Persistence. So that data or state is not lost once the containers are restarted.
* To achieve that we basically plug the physical file system path to the container’s file system path. 
* This connection of the file systems will ensure automatic replication of data.
* There are three volume types:
    1. Host Volume: **```docker run -v /home/mount/data:/var/lib/mysql/data```**
		> here the first part is the path in the host and the second part is the path in the container
    2. Anonymous Volume: **```docker run -v /var/lib/mysql/data```**
		> Here you set the path in the container, but docker decides itself the path in the host.
		For each container a folder is generated that gets mounted
    3. Named Volume: **```docker run -v name:/var/lib/mysql/data```**
		> This is an improvement on the Anonymous Volume type.
		Here you specify the name of the volume on the host directory and so you can reference the volume by name.
		This is the preferred type of volume that should be used in production.
* As above, the volumes can also be set in the docker compose yaml file, example of the mongo.yaml file using a Named Volume:
____________________________________________________________
```
    version: ‘3’
	services:
		mongodb:
			image: mongo
			ports:
			  - 27017:27017
			volumes:
			  - db-data:/var/lib/mysql/data
		mongo-express:
			image: mongo-express
			…
        # Here you list all the volumes that you want to mount into the containers. This is done at the ‘services’ level.
	# You can actually mount reference of the same folder on a host to more than one containers and that would be beneficial if those 
       # containers need to share the data
	volumes:
		db-data
```
____________________________________________________________
