# Sample Spring (Java) Based App for Deploying on Docker (and other PaaS)

## Application
The sample app is called Pet Clinic. It has been built from the following repo: https://github.com/spring-projects/spring-petclinic

If you want an updated version of the app, feel free to rebuild by cloning the above repo. The GitHub page has detailed instructions on how to do this.

## Building the Docker image
### Pre-Requisites
Docker

### Building

```
docker build --build-arg JAR_FILE=spring-petclinic-1.5.1.jar -t spring-petclinic:1.5.1 .
```
### Run

```
docker run -p 8080:8080  spring-petclinic:1.5.1
```

Verify the app is running by naviagting to http://<docker_host_IP>:8080/ on a web browser.

### Run with AppInternals Instrumentation Enabled

This assumes the recommended approach of installing an AppInternals agent on the Docker host and mounting a shared volume on the Docker container to share collected trace data. Thus, it assumes initial-mapping has been setup appropriately on the Docker host.

```
docker run -e JAVA_TOOL_OPTIONS=-agentpath:/opt/Panorama/hedzup/mn/lib/librpilj64.so \
-p 8080:8080 \
-v /opt/Panorama:/opt/Panorama \
spring-petclinic:1.5.1
```

# Deploying to Docker Swarm

### Pre-Requisites
Fully setup and configured Docker Swarm

## Building

1) SSH to Swarm Master
2) Copy Pet Clinic JAR file and Dockerfile to Master
3) Execute the build command

```
docker build --build-arg JAR_FILE=spring-petclinic-1.5.1.jar -t spring-petclinic:1.5.1 .
```

4) Run the below command to confirm the new image shows up after the build:

```
docker images
```

5) Tag the built Docker image

```
docker tag spring-petclinic:1.5.1 <master_IP>:<registry_port>/spring-petclinic:1.5.1
```

6) Push tagged image to Docker Swarm image registry
```
docker push <master_IP>:<registry_port>/spring-petclinic:1.5.1
```

## Running

Without AppInternals instrumentation:

```
docker service create  --name spring-petclinic \
-p 8080:8080 -p 9990:9990 \
<master_IP>:<registry_port>/spring-petclinic:1.5.1
```

With AppInternals instrumentation:

```
docker service create  --name spring-petclinic \
--mount type=bind,source=/opt/Panorama,destination=/opt/Panorama \
-p 8080:8080 -p 9990:9990 \
-e JAVA_TOOL_OPTIONS=-agentpath:/opt/Panorama/hedzup/mn/lib/librpilj64.so \
<master_IP>:<registry_port>/spring-petclinic:1.5.1
```
## Scaling

Scaling to the app to 5 instances:

```
docker service scale spring-petclinic=5
```

# Transaction Type Defs -- Coming Soon!
