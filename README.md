**master server**
- controls pipelines
- schedules buils

**Agents/minions**
- perform the build

**Workflow**
- a developer commits trigger to repo 
- the master server gets aware and triggers the pipeline distributing the build to agents
- agents are selected based on labels 
- agents then run the build

**Agent types**
- permanent nodes
  - dedicated to run jobs (java and ssh setup)
  - master server uses ssh to connect 
  - build tools required
- cloud agents - ephimeral/dynamic agents spun on demand
  - dockers
  - kn8
  - awsfleet manager

Build Types
- Freestyle buuld projects
  - starter shellscripts that run on server which can be triggered by events such as commit or push
- pipelines
  - use jenkins files written in groovy syntax
  - separated in stages
    - clone : pull code from repo and setup environment on agent
    - build : docker build or jar file
    - test : test the code
    - package : package or docker push
    - deploy : send artifacts to registry (sending out newly built docker image to dockerhub)

**Jenkin installation** 
https://www.jenkins.io/doc/book/installing/docker/
https://github.com/devopsjourney1/jenkins-101/blob/master/readme.md

1. Create Dockerfile with following contents
  blueocean is an addon for jenkins, makes it look a lot nicer with graphical overalys to create, 
visualize and troubleshoot CICD pipelines

```

FROM jenkins/jenkins:2.541.2-jdk21
USER root
RUN apt-get update && apt-get install -y lsb-release ca-certificates curl && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
    https://download.docker.com/linux/debian $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" \
    | tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt-get update && apt-get install -y docker-ce-cli && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow json-path-api"
```


2. Run the following command to install
```
 docker build -t myjenkins-blueocean:202602 
```
3. Create a docker network
```
docker network create myjenkinsnetwork
```
can be verfied using ```docker network ls```
4. run docker to open jenkins portal
```
docker run --name jenkins-blueocean --restart=on-failure --detach \
  --network myjenkinsnetwork --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:202602
```
5. open the portal at ```http://0.0.0.0:8080/``` or ```https://localhost:8080/```
6. Get password from ```docker exec jenkins-blueocean cat /var/jenkins_home/secrets/initialAdminPassword```
7. port forwarding for docker server (check for network to be network created)
```
docker run -d --restart=always -p 127.0.0.1:2376:2375 --network myjenkinsnetwork -v /var/run/docker.sock:/var/run/docker.sock alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock
```
```
docker inspect <container_id> | grep IPAddress
```
<IP>:2375