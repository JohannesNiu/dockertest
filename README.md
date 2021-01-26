# dockertest

Dockerfile.yml defines the makeup of the image. It's based on a base image (node:12-alpine), installs and runs the example application and its dependencies.

Build image :

  ``docker build -t dockertest .``

This image can be used to define a container. 
Run container: 

  ``docker run -dp 3000:3000 dockertest``
  
Create persistence (quick intro: https://docs.docker.com/storage/volumes/) 

  ``docker volume create test-db``
  
  ``docker run -dp 3000:3000 -v test-db:/etc/todos dockertest``
  
Bind Mounts 
  Can be used for data persistence (you get to choose precisely where your data is stored, whereas named volumes allow you to share data between containers, but Docker ultimately chooses where it puts the storage)
bind mount is used here to expose src code changes to a running container and let it react instantly, so code changes don't require stop/start of the container

   ``docker run -dp 3000:3000 -w /app -v "$(pwd):/app" node:12-alpine sh -c "yarn install && yarn run dev"``
   
Note: We changed the image from dockertest to its base image node:12-alpine. It's not yet quite clear to me why we do that :S

Multi-Container App:
"each container should do one thing and do it well"
We will separate the app from data storage and introduce a second, mysql database container. Containers that need to communicate with one another live in the same network: 

  docker network create dockertest-net
  
Start a sql server container:

  `` docker run -d
     --network todo-app --network-alias mysql
     -v todo-mysql-data:/var/lib/mysql
     -e MYSQL_ROOT_PASSWORD=secret
     -e MYSQL_DATABASE=todos
     mysql:5.7 ``

Then start our app container: 

   ``docker run -dp 3000:3000
   -w /app -v "$(pwd):/app"
   --network todo-app
   -e MYSQL_HOST=mysql
   -e MYSQL_USER=root
   -e MYSQL_PASSWORD=secret
   -e MYSQL_DB=todos
   node:12-alpine
   sh -c "yarn install && yarn run dev" ``
   
Manually starting each container is tedious, which is why we use docker-compose to automate the setup. It uses the definition from the docker-compose.yml to combine several containers into one application with only one command: 

  ``docker-compose up -d``
  
