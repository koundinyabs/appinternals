# Sample Spring (Java) Based App for Deploying on Docker (and other PaaS)

----
## Application
The sample app is called Pet Clinic. It has been built from the following repo: https://github.com/spring-projects/spring-petclinic

If you want an updated version of the app, feel free to rebuild by cloning the above repo. The GitHub page has detailed instructions on how to do this.

----

## Preparing Docker host or Docker Swarm Hosts for instrumentation
1. Install AppInternals 10.10 or later agent on Docker host or Docker Swarm Hosts
2. Download `initial-mapping` and `SVCsimple.json` to the Docker host/Docker Swarm hosts at the following location `/opt/Panorama/hedzup/mn/userdata/config` (assuming the agent is installed at /opt).

**Note**: If you are using a Mac, there are two ways to run Docker: Docker Toolbox for Mac and Docker for Mac. Docker Toolbox deploys a Linux VM as the Docker on VirtualBox. This gives you full access to the Docker host to install the AppInternals agent. Docker for Mac, on the other hand, uses HyperKit, a lightweight macOS virtualization solution built on top of Hypervisor.framework in macOS 10.10 Yosemite and higher. The HyperKit based VM does not have many of the tools the AppInternals agent installer depends on. So you may not be able to install the agent on it -- if you manage to do this, please do drop me a note.

----

## Deploying the Docker image
### Pre-Requisites
Docker host running AppInternals 10.10 agent or later (see the section [Preparing Docker host or Docker Swarm Hosts for instrumentation](#preparing-docker-host-or-docker-swarm-hosts-for-instrumentation))

### Building
1. Download `Dockerfile` and `spring-petclinic-1.5.1.jar` to the Docker host
2. Run the following command to create the PetClinic Docker image
```
docker build --build-arg JAR_FILE=spring-petclinic-1.5.1.jar -t spring-petclinic:1.5.1 .
```

Note: The same Dockerfile can be used for other Spring based applications. Just replace the reference to the jar file in the above command while building.

### Run

```
docker run -d -p 8080:8080  spring-petclinic:1.5.1
```

Verify the app is running by naviagting to http://<docker_host_IP>:8080/ on a web browser. You should see something like this:

![alt text](https://raw.githubusercontent.com/koundinyabs/appinternals/master/Docker/PetClinic.png)

### Run with AppInternals Instrumentation Enabled

#### Instrumentation Approach/Technique
This assumes the recommended approach of installing an AppInternals agent on the Docker host and mounting a shared volume on the Docker container to share collected trace data.

Further, the technique used to enable instrumentation is slightly different from the documented option of creating an instrumented image by running the createDockerFile.sh which requires installing the agent on the Docker image build machine. Instead instrumentation is enabled at run time using the `JAVA_TOOL_OPTIONS`.

Run the following command to start the Docker instance AND to have it instrumented.

```
docker run -d -e JAVA_TOOL_OPTIONS=-agentpath:/opt/Panorama/hedzup/mn/lib/librpilj64.so \
-p 8080:8080 \
-v /opt/Panorama:/opt/Panorama \
spring-petclinic:1.5.1
```

----

# Deploying to Docker Swarm

## Pre-Requisites
Fully setup and configured Docker Swarm running AppInternals 10.10 agent or later on the Swarm nodes
 (see the section [Preparing Docker host or Docker Swarm Hosts for instrumentation](#preparing-docker-host-or-docker-swarm-hosts-for-instrumentation))

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

----

# Transaction Type Defs -- Coming Soon
