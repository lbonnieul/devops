# DevOps
## TP 1

Questions :  
### 1-1 Why should we run the container with a flag -e to give the environment variables?  
It keeps the variables out of the Dockerfile preventing them to be exposed if the image is shared or pushing them into a git repo (it is hard to get rid of once pushed)

### 1-2 Why do we need a volume to be attached to our postgres container?  

By default when a container is stopped and removed, all the data is deleted, attaching a volume to our container allows the data to be persistent. The postgres data is stored into the volume in order for the next container to access it and any mofication from the previous instances of the image.

### 1-3 Document your database container essentials: commands and Dockerfile.  
Building my postgres image from the root of the repo : 
`docker build -t lbonnieul/database ./database`

Creating a network in order for our containers to communicate : `docker network create tp1-network`  

Running containers for postgres and adminer :  
`docker run -d --network tp1-network --name database -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd lbonnieul/database`  

(With the ENV command not commented in the tp1/database/Dockerfile use `docker run -d --network tp1-network --name database lbonnieul/database` instead to run the database image.)

`docker run -d --network tp1-network --name adminer_tp1 -p 8080:8080 adminer`

Adminer is now accessible at 127.0.0.1:8080  
Use the credentials passed in the -e flag of the postgres container build command and the name of the container to connect to the database

Copying the sql initialisation scripts into the container add the following lines to the Dockerfile for the database image :  
`COPY SQL-init/CreateScheme.sql /docker-entrypoint-initdb.d/CreateScheme.sql`  
`COPY SQL-init/InsertData.sql /docker-entrypoint-initdb.d/InsertData.sql`

Creating a volume for postgresql data presistence :  
`docker volume create tp1-volume`

Also need to add the flag `-v tp1-volume:/var/lib/postgresql/data` to the run command for the database-tp1 container

### 1-4 Why do we need a multistage build? And explain each step of this dockerfile.  
With multi-stage builds, you use multiple FROM statements in your Dockerfile. Each FROM instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don't want in the final image thus reducing the size of the final image. The build tools are not kept in the final image only the .jar 

### 1-5 Why do we need a reverse proxy?  
A reverse proxy hides the backend server’s IP address, reducing exposure to direct attacks. It also ease centralized routing and access control

### 1-6 Why is docker-compose so important?  
It simplifies the running of multiple-container apps by using all the Dockerfiles with on simple command

### 1-7 Document docker-compose most important commands. 

To build the images of the differents parts `docker-compose build`  
To run all the containers `docker-compose up`  
To stop the containers `docker-compose down`  

### 1-8 Document your docker-compose file.

```
services:
    backend: #the image for the java api
        build:
          context: ./backend #the path to the folder for the backend files
          dockerfile: Dockerfile #path of the backend Dockerfile
        env_file: ".env"
        environment:
        - DB_NAME=${DATABASE_NAME}
        - DB_USER=${DATABASE_USR}
        - DB_PASSWORD=${DATABASE_PWD}
        networks: #networks to be connected to (to communicate with other containers)
        - network-back-db
        - network-front-back 
        depends_on:
        - database #because the backend needs the database to be up before uping

    database: #the image for the postgres database
        build:
          context: ./database #the path to the folder for the database files
          dockerfile: Dockerfile #path of the database Dockerfile
        env_file: ".env"
        environment: #env variables
        - POSTGRES_DB=${DATABASE_NAME}
        - POSTGRES_USER=${DATABASE_USR}
        - POSTGRES_PASSWORD=${DATABASE_PWD}
        networks:
        - network-back-db #networks to be connected to (to communicate with other containers)
        volumes:
        - volume:/var/lib/postgresql/data #volume for data persistence

    frontend: #the image for the frontend (reverse proxy apache server)
        build:
          context: ./frontend #the path to the folder for the frontend files
          dockerfile: Dockerfile #path of the frontend Dockerfile
        ports: 
        - 80:80 #port mapping for apache
        networks:
        - network-front-back #networks to be connected to (to communicate with other containers)
        depends_on:
        - backend #because the frontend needs the backend to be up before uping

networks:
    network-back-db:
    network-front-back:

volumes:
    volume:

```

### 1-9 Document your publication commands and published images in dockerhub.  
Création des tag :  
`docker tag devops-frontend lbonnieul/frontend:1.0`  
`docker tag devops-backend lbonnieul/backend:1.0`  
`docker tag devops-database lbonnieul/database:1.0`  

Publication :  
`docker push lbonnieul/frontend:1.0`  
`docker push lbonnieul/backend:1.0`  
`docker push lbonnieul/database:1.0`  


### 1-10 Why do we put our images into an online repo?  
We store Docker images in online repositories for centralized access, version control, and seamless collaboration across teams and environments. This ensures secure, scalable, and automated deployment, facilitating global distribution and streamlined CI/CD workflows.