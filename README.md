# DevOps
## TP 1

Questions :  
### 1-1 Why should we run the container with a flag -e to give the environment variables?  
It keeps the variables out of the Dockerfile preventing them to be exposed if the image is shared or pushing them into a git repo (it is hard to get rid of once pushed)

### 1-2 Why do we need a volume to be attached to our postgres container?  

By default when a container is stopped and removed, all the data is deleted, attaching a volume to our container allows the data to be persistent. The postgres data is stored into the volume in order for the next container to access it and any mofication from the previous instances of the image.

### 1-3 Document your database container essentials: commands and Dockerfile.  
Building my postgres image from the root of the repo : 
`docker build -t lbonnieul/tp1/database ./tp1/database`

Creating a network in order for our containers to communicate : `docker network create tp1-network`  

Running containers for postgres and adminer :  
`docker run -d --network tp1-network --name database_tp1 -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd lbonnieul/tp1/database`  

(With the ENV command not commented in the tp1/database/Dockerfile use `docker run -d --network tp1-network --name database_tp1 lbonnieul/tp1/database` instead to run the database image.)

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

### 1-5 1-5 Why do we need a reverse proxy?  
A reverse proxy hides the backend server’s IP address, reducing exposure to direct attacks. It also ease centralized routing and access control
