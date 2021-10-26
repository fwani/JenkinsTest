

# 젠킨스 설치

- 가이드 링크 : https://www.jenkins.io/doc/book/installing/docker/

## 젠킨스 설치 (Docker)

1. 도커에서 브릿지 네트워크를 사용하기 위해서 네트워크 생성

   ```bash
   docker network create jenkins
   ```

2. jenkins docker 안에서 docker 명령어를 사용하기 위해서 `docker:dind` 이미지를 실행시킨다.

   ```bash
   docker run \
     --name jenkins-docker \
     --rm \
     --detach \
     --privileged \
     --network jenkins \
     --network-alias docker \
     --env DOCKER_TLS_CERTDIR=/certs \
     --volume jenkins-docker-certs:/certs/client \
     --volume jenkins-data:/var/jenkins_home \
     --publish 2376:2376 \
     docker:dind \
     --storage-driver overlay2
   ```

3. official jenkins 이미지를 커스터마이징 한다.

   1. Dockerfile 만들기

      ```dockerfile
      FROM jenkins/jenkins:2.303.1-jdk11
      USER root
      RUN apt-get update && apt-get install -y apt-transport-https \
             ca-certificates curl gnupg2 \
             software-properties-common
      RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
      RUN apt-key fingerprint 0EBFCD88
      RUN add-apt-repository \
             "deb [arch=amd64] https://download.docker.com/linux/debian \
             $(lsb_release -cs) stable"
      RUN apt-get update && apt-get install -y docker-ce-cli
      USER jenkins
      RUN jenkins-plugin-cli --plugins "blueocean:1.25.0 docker-workflow:1.26"
      ```

   2. 이미지 빌드하기

      ```bash
      docker build -t myjenkins-blueocean:1.1 .
      ```

4. `myjenkins-blueocean:1.1` 이미지를 실행한다.

   ```bash
   docker run \
     --name jenkins-blueocean \
     --rm \
     --detach \
     --network jenkins \
     --env DOCKER_HOST=tcp://docker:2376 \
     --env DOCKER_CERT_PATH=/certs/client \
     --env DOCKER_TLS_VERIFY=1 \
     --publish 8080:8080 \
     --publish 50000:50000 \
     --volume jenkins-data:/var/jenkins_home \
     --volume jenkins-docker-certs:/certs/client:ro \
     myjenkins-blueocean:1.1 
   ```

5. `localhost:8080` 으로 접속 후 비밀번호를 입력한다. 

   ```bash
   # 비밀번호 확인 방법
   docker logs myjenkins-blueocean:1.1
   ```

   

6. admin 세팅을 한다.

7. 시작



## 깃헙

- https://github.com/fwani/JenkinsTest
