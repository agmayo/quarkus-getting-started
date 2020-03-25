# quarkus-getting-started

It's a  microservice that returns a greeting (*hello hello!*)

The objective to achieve will be that any runtime properties defined in the file `application.properties` will override the default configuration. For this we will follow the following steps:

#### Modify the Dockerfile.native:

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
RUN microdnf install curl vi vim nano \
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

`RUN microdnf install curl vi vim nano \`
  `&& microdnf update \`
  `&& microdnf clean all` 

By placing an `application.properties` file inside a directory named `config` which resides in the directory where the application runs, any runtime properties defined in that file will override the default configuration.

#### Generate a docker image

Execute:

```bash
mvn package -Pnative -Dquarkus.native.container-runtime=docker
```

#### Create the docker image in the image local repository:

```bash
docker build -f src/main/docker/Dockerfile.native -t agmayo/testing-external-properties .
```

#### Run the image's container:

```bash
docker run -i -p 8080:8080 agmayo/testing-external-properties
```



## Testing

**Content of `application.properties:`**

```properties
# Configuration file
# key = value
greeting.message = hello
greeting.name = hello
```

To check it's behavior just run the following `curl`:

```bash
curl http://localhost:8080/hello
```

It will return:

`hello hello!`

We will also rename the `.dockerignore` file. In this case we will do:

```bash
mv .dockerignore kk 
```

To verify that the service changes we will go into the container and edit the file `application.properties`:

```bash
docker exec -it <CONTAINER_ID> bash   
```

**Content of `application.properties:`**

```properties
# Configuration file
# key = value
greeting.message = bye
greeting.name = bye
```

To check it's behavior just run the following `curl`:

```bash
curl http://localhost:8080/hello
```

It will return:

`bye bye!`



# Docker-compose 

In the next step we will create a docker-compose in which we have a service and a volume that maps a local directory to a directory of the container.

### Folder structure:

```
├── microservice-A
│    └── application.properties
├── docker-compose.yml
```



### Content of the docker-compose.yml:

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

### Building and running

To install the microservice you'll have to:

* Sing in `docker-hub` with : 

  ```bash
  docker login
  # Fill up the credentials with an user with permissions.
  ```

* Download the image:

  ```bash
  docker push agmayo/testing-external-properties:latest
  ```

* Build all docker images

  ```bash
  docker-compose build
  ```

* Starts the containers in the background and leaves them running:

  ```bash
  docker-compose up -d
  ```



## Testing

To check it's behavior just run the following `curl`:

```bash
curl http://localhost:8080/hello
```

It will return:

`hello hello!`



- Stops containers:

```bash
docker-compose down
```

- Starts the containers in the background and leaves them running:

```bash
docker-compose up -d
```



We will change the file `application.properties` and verify that the response has also changed.

To check it's behavior just run the following `curl`:

```bash
curl http://localhost:8080/hello
```

It will return:

`bye bye!`

