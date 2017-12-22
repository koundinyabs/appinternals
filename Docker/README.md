# Sample Spring (Java) Based App for Deploying on Docker (and other PaaS)

## Application
The sample app is called Pet Clinic. It has been built from the following repo: https://github.com/spring-projects/spring-petclinic

If you want an updated version of the app, feel free to rebuild by cloning the above repo. The GitHub page has detailed instructions on how to do this.

## Building the Docker image
### Pre-Requisites
Docker (for building)

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

### Transaction Type Defs -- Coming
