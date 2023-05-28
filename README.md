# devsecops-class-docker

### Pre requisiteis

- launch ubuntu ec2
- Perform the following steps in ubuntu ec2

## Update OS

- Ref : https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04

```
sudo apt update -y

sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y

apt-cache policy docker-ce

sudo apt install docker-ce -y

sudo systemctl status docker

```

- check docker version: docker version
- check docker commands: docker

## Create Docker Volume for Data and Log

```
sudo docker volume create jenkins-data
sudo docker volume create jenkins-log
sudo docker volume ls
```

## Install Jenkins with Docker

1. create docker folder
2. Create Dockerfile

```
FROM jenkins/jenkins
LABEL maintainer="partha.abdas@gmail.com"
USER root
RUN mkdir /var/log/jenkins
RUN mkdir /var/cache/jenkins
RUN chown -R jenkins:jenkins /var/log/jenkins
RUN chown -R jenkins:jenkins /var/cache/jenkins
USER jenkins

ENV JAVA_OPTS="-Xmx8192m"
ENV JENKINS_OPTS="--logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war"
```

## Build the docker image

```
cd docker
sudo docker build -t <image-name>:1.0 .
sudo docker images
```

## Run Jenkins Container with Data and Log Volume

```
sudo docker run -p 8080:8080 -p 50000:50000 \
    --name=jenkins-master \
    --mount source=jenkins-log,target=/var/log/jenkins \
    --mount source=jenkins-data,target=/var/jenkins_home \
    -d <image-name>:1.0

sudo docker ps
```

## Login to the container

```
sudo docker exec jenkins-master tail -f /var/log/jenkins/jenkins.log
```

## Access jenkins

http://public-ip:8080

### Push to DockerHub

- docker login
- sudo docker tag devsecops-jenkins:1.0 partha2019/devsecops-jenkins:1.0
- sudo docker push partha2019/devsecops-jenkins:1.0
- Please make sure the uploaded images exists in dockerhub

### Download to another Instance

- launch another ubuntu machine and install docker using above commands
- Create Docker Volume for Data and Log

```
sudo docker volume create jenkins-data
sudo docker volume create jenkins-log
sudo docker volume ls
```

- get the image

```
docker pull partha2019/devsecops-jenkins:1.0
```

- run the image

```
sudo docker run -p 8080:8080 -p 50000:50000  --name=devsecops-may2023  -d partha2019/devsecops-may2023:1.0
```

- Access Jenkins: http://public-ip:8080

## DevSecOps

- Reference: https://container-devsecops.awssecworkshops.com

### Dockerfile linitng

- Hadolint
- https://github.com/hadolint/hadolint
- sudo docker run --rm -i -v ${PWD}/hadolint.yml:/.hadolint.yaml hadolint/hadolint:v1.16.2 hadolint -f json - < Dockerfile

### Secrets scanning

- trufflehog
- https://github.com/trufflesecurity/trufflehog
- sudo docker run --rm -it -v "$PWD:/pwd" trufflesecurity/trufflehog:latest github --repo https://github.com/trufflesecurity/test_keys

### Vulnerabity scanning

- Anchore
- https://docs.anchore.com/current/docs/using/cli_usage/images/
- sudo curl -sSfL https://anchorectl-releases.anchore.io/anchorectl/install.sh | sudo sh -s -- -b /usr/local/bin
