# jenkins
1. [Install Jenkins Use Docker](#Install-Jenkins-Use-Docker)
2. [Integrate Jenkins with Github](#Integrate-Jenkins-with-Github)
3. [Setup Auto Deploy Use Jenkins](#Setup-Auto-Deploy-Use-Jenkins)

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
  --name jenkins-docker \
  --restart=unless-stopped \
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
  jenkins/jenkins:jdk17 \
  ```

copy the password
  ```bash
  docker logs jenkins-docker
  ```

goto web browser
  ```bash
  http://localhost:8080
  ```

### Integrate Jenkins with Github

Add Plugins
```bash
Docker
Docker Commons
Docker Pipeline
Docker API
Github Integration
```

Set up SSH key
```bash
docker exec -it jenkins-docker bash
```
```bash
ssh-keygen -t rsa -b 4096 -C "your_email_address"
```
```bash
cat ~/.ssh/id_rsa.pub
```
goto https://github.com/settings/keys and add the public key

test the connection
```bash
ssh -T git@github.com
# Hi ******! You've successfully authenticated, but GitHub does not provide shell access.
```

goto https://github.com/settings/tokens/new and get token 
goto http://localhost:8080/manage/credentials/store/system/domain/_/newCredentials
```bash
Kind : secret text
Scope : Global
Secret : your token
id : 
Description : github token
```

setup Github Server
goto http://localhost:8080/manage/configure
```bash
# Add GitHub Servers
Name : github
API URL : https://api.github.com
Credentials : github token
click test connection
# Credentials verified for user asmitul, rate limit: 4999
```

### Setup Auto Deploy Use Jenkins
repo > settings > webhooks > add 
```bash
http://localhost:8080/github-webhook/
# Content type
application/json
select Just the push event. 
```

copy repo ssh adress

goto Jinkins 
```bash
# 1. new Job
# 2. Free Style
# 3. Source Code Management : Git
# 4. Repository URL : https://github.com/your_username/your_repo_name.git
# 5. Branch : main
# 6. Build Triggers : GitHub hook trigger for GITScm polling
# 7. Build Steps : Run shell script
```

```bash
#!/bin/bash
set -e

CONTAINER_NAME="container_name"

# Stop and remove the container if it exists
if docker ps -a --format '{{.Names}}' | grep "$CONTAINER_NAME"; then
    docker stop "$CONTAINER_NAME"
    docker rm -f "$CONTAINER_NAME"
fi

# Remove the image if it exists
if docker images --format '{{.Repository}}' | grep "$CONTAINER_NAME"; then
    docker rmi -f "$CONTAINER_NAME"
fi

# Build the new image
if docker build -t "$CONTAINER_NAME" .; then
    echo "Image built successfully."
else
    echo "Failed to build the image."
    exit 1
fi

# Deploy the new container
if docker run -d --restart always --name "$CONTAINER_NAME" "$CONTAINER_NAME" python /code/app/main.py; then
    echo "Container deployed successfully."
else
    echo "Failed to deploy the container."
    exit 1
fi

# Clean up stopped containers and unused images
docker container prune -f
docker image prune -f
```