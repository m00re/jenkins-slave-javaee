# Jenkins-Slave Docker-Image

A Jenkins-Slave Docker image including OpenJDK 1.8.0, Gradle, Git and Docker CE.

## Available Docker Images at DockerHub

Image Name                    | Tag   | Jenkins Swarm | OpenJDK     | Docker CE | Git     | Gradle
------------------------------|-------|---------------|-------------|-----------|---------|---------
m00re/jenkins-slave-javaee    | 3.6   | 3.6           | 1.8.0.212   | 18.09.6   | 1.8.3.1 | 4.6

See: https://hub.docker.com/r/m00re/jenkins-slave-javaee/

## Building

Simply type ```docker build . -t <YourTagName>``` to rebuild the image yourself. However, you do not necessarily need to rebuild the image yourself, I provide pre-built images at https://hub.docker.com/r/m00re/jenkins-slave-hashicorp/.

If you want to rebuild the image using different versions of Virtualbox, Packer, Vagrant or Swarm, you can do so by overriding the following build arguments:
- ```GRADLE_VERSION```: by default set to ```4.6```
- ```SWARM_VERSION```: by default set to ```3.6``` (you'll also need to set the proper SHA1 checksum in ```SWARM_SHA```).

## Usage

Simply type

```
docker run \
  -e "SWARM_VM_PARAMETERS=" \
  -e "SWARM_MASTER_URL=http://yourjenkinsmasterurl:8080/" \
  -e "SWARM_VM_PARAMETERS=" \
  -e "SWARM_JENKINS_USER=slave" \
  -e "SWARM_JENKINS_PASSWORD=slave" \
  -e "SWARM_CLIENT_EXECUTORS=2" \
  -e "SWARM_CLIENT_LABELS=openjdk gradle git docker" \
  -e "SWARM_CLIENT_NAME=" \
  -v /var/run/docker.sock:/var/run/docker.sock \
  m00re/jenkins-slave-javaee:3.6
```

to spawn a new Jenkins slave docker container with ```2 executors```, Jenkins labels ```openjdk```, ```gradle```, ```git``` and ```docker```, using the Jenkins credentials ```slave/slave```, and connecting to a Jenkins master instance running at ```http://yourjenkinsmasterurl:8080/```.

> **NOTE: Please be aware that the above example uses HTTP to connect your slave node to the master node. This is not recommended. Instead, use a setup as provided below, in which master and slave nodes are linked over a virtual private Docker network.**

## Usage example based on ```docker-compose.yml```

The following example starts up a Jenkins master node and a Jenkins slave node in an own Docker network. The Jenkins slave plugin ensures successful auto-discovery of the master node by the slave node. This example uses the jenkins-docker image defined in https://github.com/m00re/jenkins-docker.

```
version: '2'
services:

  # Jenkins Master
  jenkins:
    image: m00re/jenkins-docker:2.178-alpine 
    container_name: jenkins
    hostname: jenkins
    networks:
      - jenkins
    ports:
     - "8080:8080"
    volumes:
      - jenkinsdata:/var/jenkins_home
    environment:
      - JAVA_VM_PARAMETERS=-Xmx1024m -Xms512m
      - JENKINS_PARAMETERS=
      - JENKINS_MASTER_EXECUTORS=0
      - JENKINS_SLAVEPORT=50000
      - JENKINS_PLUGINS=
      - JENKINS_ADMIN_USER=admin
      - JENKINS_ADMIN_PASSWORD=test
      - JENKINS_KEYSTORE_PASSWORD=
      - JENKINS_LOG_FILE=
      - JENKINS_USER_NAMES=slave
      - JENKINS_USER_PERMISSIONS=jenkins.model.Jenkins.READ:hudson.model.Computer.CONNECT:hudson.model.Computer.DISCONNECT:hudson.model.Computer.CREATE
      - JENKINS_USER_PASSWORDS=slave

  # Jenkins Slave
  slave:
    image: m00re/jenkins-slave-hashicorp:3.6
    networks:
      - jenkins
    environment:
      - SWARM_VM_PARAMETERS=
      - SWARM_MASTER_URL=
      - SWARM_CLIENT_PARAMETERS=
      - SWARM_JENKINS_USER=slave
      - SWARM_JENKINS_PASSWORD=slave
      - SWARM_CLIENT_EXECUTORS=1
      - SWARM_CLIENT_LABELS=openjdk gradle git docker
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

volumes:
  jenkinsdata:
    external: false

networks:
  jenkins:
    driver: bridge
```

## Acknowledgements

A huge thank you goes to [blacklabelops](https://github.com/blacklabelops/) for his Dockerfile recipes in https://github.com/blacklabelops/swarm/tree/master/hashicorp-virtualbox and https://github.com/blacklabelops/jenkins-swarm. His scripts served as the starting point for the above image.
