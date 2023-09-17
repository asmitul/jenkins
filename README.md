# jenkins
1. [Install Jenkins Use Docker](#Install-Jenkins-Use-Docker)

### Install Jenkins Use Docker
Create a bridge network in Docker using the following docker network create command:
```bash
docker network create jenkins
```
```bash
docker pull jenkins/jenkins:jdk17
```
```bash
docker run \
  --name jenkins \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  docker:dind \
  --storage-driver overlay2
  ```