1) docker pull jenkins/jenkins:lts
2) useradd jenkins
3) passwd jenkins
4) groupadd jenkins
5) usermod -aG wheel jenkins
6) cd /home/jenkins
7) mkdir /home/jenkins/jenkins-data 
8) cd /home/jenkins/jenkins-data
9) mkdir home/jenkins/jenkins-data/jenkins_home
10) touch docker-compose.yml

===============================================================
{
version: '3'
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins
    ports:
      - "8080:8080"
    volumes:
      - $PWD/jenkins_home:/var/jenkins_home
    networks:
      - net
networks:
  net:
} >> docker-compose.yml

================================================================

11) chmod +777 /home/jenkins/jenkins-data/jenkins_home
12) chown jenkins:jenkins /home/jenkins/jenkins-data/jenkins_home -R
13) docker-compose up -d


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>



1) cd into jenkins folder and create "centos7" folder
2) cd centos7
3) vim Dockerfile and write them

================================================================================

FROM centos:7

RUN yum -y install openssh-server
RUN yum -y install passwd

RUN useradd remote_user && \
    echo "123456" | passwd remote_user  --stdin && \
    mkdir /home/remote_user/.ssh && \
    chmod 700 /home/remote_user/.ssh

COPY remote-key.pub /home/remote_user/.ssh/authorized_keys

RUN chown remote_user:remote_user   -R /home/remote_user && \
    chmod 600 /home/remote_user/.ssh/authorized_keys


RUN yum install -y initscripts && \
    /usr/sbin/sshd-keygen

CMD /usr/sbin/sshd -D

================================================================================

4) then write command: ssh-keygen -f remote-key
5) cd ..
6) vim docker-compose.yml
7) write into docker-compose.yml

================================================================================

version: '3'
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins
    ports:
      - "8080:8080"
    volumes:
      - $PWD/jenkins_home:/var/jenkins_home
    networks:
      - net
  remote_host:
    container_name: remote-host
    image: remote-host
    build:
      context: centos7
    networks:
      - net
networks:
  net:

================================================================================

8) pwd >> /home/jenkins/jenkins-data
9) docker-compose build
10) docker-compose up -d
11) docker exec -it jenkins bash
12) (write into jenkins's bash) ssh remote_user@remote_host >>enter >> yes >> write pass
13) artiq jenkins ve centos-7 serverleri bir birlerini gorur ve aralarinda connection successfuldur ve esasi ise bunlarin her ikisi bir folderin altinda olmali,bir docker-compose un altinda birleshmish formadadir
14) Then go to the Jenkins in web and download ssh plugin
15) Go to Manage Jenkins >> Manage credential >> write credential with private keys >> then go to configure system >> ssh remote hosts >> remote_host >> 22 >> and select credential >> new item >> freestylejob >> build >> execute shell script on remote host using ssh >> then write command >> apply and save
