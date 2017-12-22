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
docker run -p 8080:8080 mysql:5.7.8 spring-petclinic:1.5.1
```

### Run with AppInternals Instrumentation Enabled

```
docker run -e JAVA_TOOL_OPTIONS=-agentpath:/opt/Panorama/hedzup/mn/lib/librpilj64.so \
-p 8080:8080 mysql:5.7.8 spring-petclinic:1.5.1
```
