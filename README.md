# korkenxyz-infrastructure
- Terraform
- S3
- CloudFront
- Travis CI

1. Change the following into your own domains before running `terraform apply`.
```
# Variable for domain name
variable "www_domain_name" {
  default = "www.korken.xyz"
}

# Variable for root domain
variable "root_domain_name" {
  default = "korken.xyz"
}
```

2. Make sure that you have access to one of the following e-mail addresses associated with your domain. I personally registered a free Zoho account that allowed me to create an e-mail address for my domains purchased from GoDaddy.

    - administrator@your_domain_name
    - hostmaster@your_domain_name
    - postmaster@your_domain_name
    - webmaster@your_domain_name
    - admin@your_domain_name

Once you have setup an e-mail address with your domain, run `terraform apply` and wait for 2 e-mails. Confirm those e-mails and run `terraform apply` again.

3. You should now have the certificate ready! Wait for the DNS to propagate properly and you should have your domain working shortly. One important thing to keep in mind is that CloudFront only generate certificates for us-east-1 region.





# SAPS - Docker documentation

## Table of Contents
- [Dockerfile](#dockerfile)
- [.dockerignore](#dockerignore)
- [docker-compose.yml](#docker-compose)
- [Build and run app locally](#Build-and-run-app-locally)
- [Build and run image locally](#Build-and-run-image-locally)
- [Docker commands](#Docker-commands)


* Dockerfile
* .dockerignore
* docker-compose.yml
* Build & Run App locally
* Build and run image locally
* Docker commands

## Dockerfile
Docker uses the `Dockerfile` to build images automatically.  
* `FROM node:` specifies what type of image we want to build.
* `WORKDIR /usr/src/app` creates the directory to hold code for application.
* `COPY package*.json ./` copy package.json AND package-lock.json.
* `RUN npm install` will install all modules listed as dependencies.
* `COPY . .` bundle source code.
* `EXPOSE 8080` binds the app to port 8080.
* `CMD [ "npm", "start" ]` start app.

```
FROM node:

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm install --only=production

# Bundle app source
COPY . .

EXPOSE 8080
CMD [ "npm", "start" ]
```

## .dockerignore
The `.dockerignore` file prevent local modules and debug logs from being copied onto Docker image.

```
node_modules
npm-debug.log
```

## docker-compose.yml
Docker compose allows us to run multi-container Docker applications.
* `version: '3.3'` refers to which compose file format that's being used. 3.3 uses the 17.06.0+ Docker Engine.

    **node Service**
*   `build: context: .` path to Dockerfile.
* `ports: - "3000:3000"` maps ports (HOST:CONTAINER).
* `depnds_on: - mysql` express dependency between service.
* `networks: - docker_network` which network to join.
* `volumes: - .:/usr/src/app and - /usr/src/app/node_modules` mounted host path and module.

  **MySQL Service**
* `build: context: ./mysql` path to MySQL directory.
* MYSQL credentials are passed as environment variables.
```
args:
  - MYSQL_DATABASE=default_database
  - MYSQL_USER=default_user
  - MYSQL_PASSWORD=secret
  - MYSQL_ROOT_PASSWORD=root
```
* `volumes: - ./data/mysql/:/var/lib/mysql` mount host path.
* `ports: - "3307:3306"` maps ports(HOST:CONTAINER)
* `networks: - docker_network` which network to join.

  **Network**
* `driver: bridge` creates a private internal network on the host so that containers are able to communicate with each other.
```
  networks:
      docker_network:
          driver: bridge
```

**docker-compose.yml**
```
version: '3.3'

services:
    node:
        build:
            context: .
        ports:
            - "3000:3000"
        depends_on:
            - mysql
        networks:
            - docker_network
        volumes:
            - .:/usr/src/app
            - /usr/src/app/node_modules
        command: npm run dev
    mysql:
        build:
            context: ./mysql
            args:
                - MYSQL_DATABASE=default_database
                - MYSQL_USER=default_user
                - MYSQL_PASSWORD=secret
                - MYSQL_ROOT_PASSWORD=root
        volumes:
            - ./data/mysql/:/var/lib/mysql
        ports:
            - "3307:3306"
        networks:
            - docker_network
networks:
    docker_network:
        driver: bridge

```

## Build & Run App locally
1. Run `docker build -t saps-app .`  	
2. It will take some time to install the dependencies, be patient.
3. Once done, run `docker images` and you will see your newly created image there.
4. Run `docker run -p 3000:8080 -d saps-app` which will map port 8080 inside of the container to port 3000 your machine.
5. Run `curl -i localhost:3000` or visit http://localhost:3000/ (\ No newline at end of file)

## Build & Run image locally
1. Run `docker-compose build` to build the docker images
2. Run `docker-compose up` to START the containers
3. Run `docker-compose down` to STOP the containers
4. Run `curl -i localhost:3000` or visit http://localhost:3000/

This will start a Node.JS instance with Express.js and a MySQL DB which the Express.js server can connect to.

## Docker commands
- `docker ps` = List containers.
- `docker build` = Build image from Dockerfile
- `docker images` = List images.
- `docker ps -aq` = List all containers.
- `docker stop $(docker ps -aq)` = Stop all running containers.
- `docker rm $(docker ps -aq)` = Remove all containers.
- `docker rmi $(docker images -q)` = Remove all images.
