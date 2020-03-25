# quarkus-getting-started

The objective to achieve will be that any runtime properties defined in the file `application.properties` will override the default configuration. This is achieved by placing an `application.properties` file inside a directory named `config` which resides in the directory where the application runs. For this we will work with a microservice that returns a greeting (*hello hello!*).

To complete the above we will make use of a Compose, which is basically a three-step process.

1. Define your environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the services  in `docker-compose.yml` so they can be run together in an isolated environment.
3. Lastly, run `docker-compose up`.

## Getting started

We have followed the following steps:

#####  Modify the Dockerfile.native:

```
####
# This Dockerfile is used in order to build a container that runs the Quarkus application in native (no JVM) mode
#
# Before building the docker image run:
#
# mvn package -Pnative -Dquarkus.native.container-build=true
#
# Then, build the image with:
#
# docker build -f src/main/docker/Dockerfile.native -t quarkus/quarkus-getting-started .
#
# Then run the container using:
#
# docker run -i --rm -p 8080:8080 quarkus/quarkus-getting-started
#
###
FROM registry.access.redhat.com/ubi8/ubi-minimal:8.1
WORKDIR /work/

# installing text editors for properties update
RUN microdnf install vi vim nano \
  && microdnf update \
  && microdnf clean all 
  
COPY target/*-runner /work/application

# Copying application.properties in container
COPY src/main/resources/application.properties /work/config/

# set up permissions for user `1001`
RUN chmod 775 /work /work/application \
  && chown -R 1001 /work \
  && chmod -R "g+rwX" /work \
  && chown -R 1001:root /work


EXPOSE 8080
USER 1001

CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
```



What has been added in this file is:

`COPY src/main/resources/application.properties /work/config/`

`RUN microdnf install vi vim nano \`
  `&& microdnf update \`
  `&& microdnf clean all` 

In addition, we also delete the file: `.dockerignore`

###### Content of .dockerignore:

```
*
!target/*-runner
!target/*-runner.jar
!target/lib/*
```

We are going to do this so that it can be run: `COPY src / main / resources / application.properties / work / config /`

##### Docker-compose

In the next step we will create a docker-compose in which we have a service and a volume that maps a local directory to a directory of the container.

###### Content of the docker-compose.yml:

```yml
version: "3.7"
services:
  testing-external-properties:
    image: testing-external-properties
    build:
      context: .
      dockerfile: ./src/main/docker/Dockerfile.native
    container_name: testing-external-properties
    ports: 
        # To receive the curls
        - 8080:8080
    volumes: 
        # Volume for  persistance
        - type: bind
          source: ./src/main/resources/application.properties
          target: /work/config/application.properties
```



A bind mount is a file or folder stored anywhere on the container host filesystem, mounted into a running container.



## Folder structure:

```
├── microservice-A
│    └── application.properties
├── docker-compose.yml
```



## Testing

To install the microservice you'll have to:

1. Starts the containers in the background and leaves them running:


```bash
docker-compose up -d
```

​	**Content of `application.properties:`**

```properties
# Configuration file
# key = value
greeting.message = hello
greeting.name = hello
```

2. To check it's behavior just run the following `curl`:

```
curl http://localhost:8080/hello 
```

​	It will return:

​	`hello hello!`

3. Stops containers:

```bash
docker-compose down
```

4. We will change the file `application.properties` and verify that the response has also changed:

**Content of `application.properties:`**

```properties
# Configuration file
# key = value
greeting.message = bye
greeting.name = bye
```

5. Starts the containers in the background and leaves them running:

```bash
docker-compose up -d
```

6. To check it's behavior just run the following `curl`:

```bash
curl http://localhost:8080/hello
```

​	It will return:

​	`bye bye!`

## Clean up

Don't leave without cleaning your machine!

```bash
docker-compose down --volumes --rmi all --remove-orphans
```



