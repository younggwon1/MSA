# CI/CD

[virtualbox 다운로드](https://www.virtualbox.org/)

[vagrant 다운로드](https://www.vagrantup.com/)

> virtualbox를 사용하기 위해서는 docker를 중단하고 실행한다.
>
> hypervisor off 확인 (관리자 명령 cmd)
>
> ```
> C:\Windows\system32>bcdedit
> ```
>
> ```
> C:\Windows\system32>bcdedit /set hypervisorlaunchtype off
> ```
>
> Vagrantfile에 public_network -> private_network로 변경한 후 cmd창에 vagrant halt한 다음 vagrant up 해준다.

![image](https://user-images.githubusercontent.com/42603919/82625359-ddf59580-9c1f-11ea-9050-924b93ca08c7.png)



```powershell
C:\Users\HPE>vagrant plugin install vagrant-vbguest
```



##### 다음에 해당하는 디렉토리로 이동한 후 다음 명령어를 실행.

```powershell
C:\Users\HPE\work\vagrant>vagrant up
```



##### 설치 완료 후 jenkins-server에 접속해보기

```powershell
C:\Users\HPE\work\vagrant>vagrant ssh jenkins-server
[vagrant@jenkins-server ~]$
```



##### jenkins-server에 java 설치하기

```powershell
[vagrant@jenkins-server ~]$sudo yum -y install java-1.8*
```

```powershell
[vagrant@jenkins-server ~]$find /usr/lib/jvm/java-1.8* | head -n 3
/usr/lib/jvm/java-1.8.0
/usr/lib/jvm/java-1.8.0-openjdk
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64
```

- bash_profile vi 편집기에 들어가기

```powershell
[vagrant@jenkins-server ~]$ vi ~/.bash_profile
```

- bash_profile vi 편집기에서 작업

```
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-<Java version which seen in the above output>
export JAVA_HOME
PATH=$PATH:$JAVA_HOME (PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME)

shift + : => wq!

[vagrant@jenkins-server ~]$ exit

C:\Users\HPE\work\vagrant> vagrant ssh jenkins-server (다시 로그인)
```





##### jenkins 설치하기 (jenkins-server에서)

```powershell
[vagrant@jenkins-server ~]$ sudo yum -y install wget
```

```powershell
[vagrant@jenkins-server ~]$ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

```powershell
[vagrant@jenkins-server ~]$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

```powershell
[vagrant@jenkins-server ~]$ sudo yum -y install jenkins
```



##### jenkins 시작

```powershell
[vagrant@jenkins-server ~]$ sudo service jenkins start
```

```powershell
[vagrant@jenkins-server ~]$ sudo chkconfig jenkins on
```

```powershell
[vagrant@jenkins-server ~]$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![image](https://user-images.githubusercontent.com/42603919/82849201-cc6b0100-9f31-11ea-9aa6-b236f9b49ae8.png)



![image](https://user-images.githubusercontent.com/42603919/82633942-7c412580-9c37-11ea-9a0a-99333c6b50bf.png)



---



##### 설치 완료 후 tomcat-server에 접속해보기

```powershell
C:\Users\HPE\work\vagrant>vagrant ssh tomcat-server
[vagrant@tomcat-server ~]$
```



##### tomcat-server에 java 설치하기

```powershell
[vagrant@tomcat-server ~]$sudo yum -y install java-1.8*
```

```powershell
[vagrant@tomcat-server ~]$find /usr/lib/jvm/java-1.8* | head -n 3
/usr/lib/jvm/java-1.8.0
/usr/lib/jvm/java-1.8.0-openjdk
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64
```

- bash_profile vi 편집기에 들어가기

```powershell
[vagrant@tomcat-server ~]$ vi ~/.bash_profile
```

- bash_profile vi 편집기에서 작업

```
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-<Java version which seen in the above output>
export JAVA_HOME
PATH=$PATH:$JAVA_HOME (PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME)

shift + : => wq!

[vagrant@tomcat-server ~]$ exit

C:\Users\HPE\work\vagrant> vagrant ssh tomcat-server (다시 로그인)
```



##### apache-tomcat 설치하기 (tomcat-server에서)

```powershell
[vagrant@tomcat-server ~]$ sudo yum -y install wget
```

```powershell
[vagrant@tomcat-server ~]$ sudo wget http://mirror.navercorp.com/apache/tomcat/tomcat-9/v9.0.35/bin/apache-tomcat-9.0.35
.tar.gz
```

```powershell
[vagrant@tomcat-server ~]$ sudo mkdir -p /usr/local/tomcat
```

```powershell
[vagrant@tomcat-server ~]$ sudo mv apache-tomcat-9.0.35.tar.gz /usr/local/tomcat
```

```powershell
[vagrant@tomcat-server ~]$ cd /usr/local/tomcat
```

```powershell
[vagrant@tomcat-server tomcat]$ sudo tar -xvzf apache-tomcat-9.0.35.tar.gz
```



##### apache-tomcat 시작

```powershell
[vagrant@tomcat-server tomcat]$ cd apache-tomcat-9.0.35/
```

```powershell
[vagrant@tomcat-server apache-tomcat-9.0.35]$ sudo ./bin/startup.sh
```



![image](https://user-images.githubusercontent.com/42603919/82642345-b49d2f80-9c48-11ea-9bdd-902e9ce2bb50.png)



##### Tomcat Manager 설정 (403 Error 해결)

1. **tomcat-users.xml** 수정

- sudo find ./ -name tomcat-users.xml (파일 위치 찾기)

```powershell
[vagrant@tomcat-server ~]$ cd /usr/local/tomcat/apache-tomcat-9.0.35
```

```powershell
[vagrant@tomcat-server apache-tomcat-9.0.35]$ sudo vi conf/tomcat-users.xml
```

```xml
# 추가

<tomcat-users>
  <role rolename="manager-gui"/>
  <role rolename="manager-jmx"/>
  <role rolename="manager-script"/>
  <role rolename="manager-status"/>
  <user username="admin" password="admin" roles="manager-gui,manager-jmx,manager-script,manager-status"/>
  <user username="deployer" password="deployer" roles="manager-script"/>
  <user username="tomcat" password="tomcat" roles="manager-gui"/>
</tomcat-users>
```



2. **context.xml** 수정

- sudo find ./ -name context.xml (파일 위치 찾기)

```powershell
[vagrant@tomcat-server apache-tomcat-9.0.35]$ sudo vi /usr/local/tomcat/apache-tomcat-9.0.35/webapps/manager/META-INF/context.xml
```

```xml
<Context antiResourceLocking="false" privileged="true" >
<!--  
         <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
```



http://localhost:28080/manager/ 로 접속하면 **Tomcat 웹 애플리케이션 매니저**가 보이게 된다.



---



##### jenkins와 tomcat server 띄우기

```powershell
C:\Users\HPE\work\vagrant>vagrant ssh jenkins-server
[vagrant@jenkins-server ~]$
```

```powershell
C:\Users\HPE\work\vagrant>vagrant ssh tomcat-server
[vagrant@tomcat-server ~]$
```



##### Server간 접속 연동하기 (다른 host로 접속)

```powershell
[vagrant@docker-server ~]$ sudo vi /etc/ssh/sshd_config
[vagrant@jenkins-server ~]$ sudo vi /etc/ssh/sshd_config
[vagrant@tomcat-server ~]$ sudo vi /etc/ssh/sshd_config
```

![image](https://user-images.githubusercontent.com/42603919/82635605-7d745180-9c3b-11ea-88ae-64954f583238.png)

```powershell
[vagrant@docker-server ~]$ sudo systemctl restart sshd
[vagrant@jenkins-server ~]$ sudo systemctl restart sshd
[vagrant@tomcat-server ~]$ sudo systemctl restart sshd
```

- 이름으로 접속할 수 있도록 설정

```
sudo vi /etc/hosts

# 각각의 Host에 자신을 제외한 나머지를 입력한다.
192.168.56.11 jenkins-server

192.168.56.12 tomcat-server

192.168.56.13 docker-server
```



##### 업로드 작업(배포 작업)

1. github에 작업할 repository를 생성

<img src="https://user-images.githubusercontent.com/42603919/82782751-0edfff80-9e98-11ea-9c0a-08b344b1677e.png" alt="image" style="zoom:67%;" />



2. 새로운 item 만들기 (jenkins)

   - jenkins관리 -> 플러그인 설치 (**github(github, integration), maven(invoker, integration), Deploy to(deploy to container)**) -> 재시작없이 설치



##### jenkins, tomcat server에 git 설치하기

```powershell
[vagrant@jenkins-server ~]$ sudo yum install git -y
[vagrant@tomcat-server ~]$ sudo yum install git -y
```



##### jenkins, tomcat server에 maven 설치하기

```powershell
[vagrant@jenkins-server ~]$ sudo yum install maven -y
[vagrant@tomcat-server ~]$ sudo yum install maven -y
```



##### Git, add maven

- Global tool configuration

![image](https://user-images.githubusercontent.com/42603919/82849980-6f714a00-9f35-11ea-9426-53d9c32b3eaf.png)



![image](https://user-images.githubusercontent.com/42603919/82850389-2fab6200-9f37-11ea-9ab5-46823a8deb7e.png)

##### git과 maven이 설치됨



---



##### [Maven Project 시작]

![image-20200526095744344](C:\Users\HPE\AppData\Roaming\Typora\typora-user-images\image-20200526095744344.png)

- 소스 코드 관리

![image](https://user-images.githubusercontent.com/42603919/82850543-cf68f000-9f37-11ea-8120-ec8b73fd80ed.png)

- Build

![image](https://user-images.githubusercontent.com/42603919/82850742-84031180-9f38-11ea-84fa-1e8ee3bfa566.png)

##### Apply -> 저장 ->  -> Build Now



##### github에 올라가 있는 project를 빌드하여 다음과 같이 보인다.

```powershell
[vagrant@jenkins-server ~]$ cd /var/lib/jenkins/workspace/My_First_Maven_Build
[vagrant@jenkins-server My_First_Maven_Build]$ ls -al
total 16
drwxr-xr-x. 5 jenkins jenkins   96 May 26 01:02 .
drwxr-xr-x. 4 jenkins jenkins   58 May 26 01:02 ..
-rw-r--r--. 1 jenkins jenkins  202 May 26 01:02 Dockerfile
drwxr-xr-x. 8 jenkins jenkins  162 May 26 01:06 .git
-rw-r--r--. 1 jenkins jenkins 5969 May 26 01:02 pom.xml
-rw-r--r--. 1 jenkins jenkins   38 May 26 01:02 README.md
drwxr-xr-x. 4 jenkins jenkins   46 May 26 01:07 server
drwxr-xr-x. 4 jenkins jenkins   46 May 26 01:07 webapp
```



##### war,jar 파일이 있으면 project build를 할 수 있다.

```powershell
[vagrant@jenkins-server My_First_Maven_Build]$ cd webapp/target
[vagrant@jenkins-server target]$ ls -al
total 4
drwxr-xr-x. 5 jenkins jenkins   76 May 26 01:08 .
drwxr-xr-x. 4 jenkins jenkins   46 May 26 01:07 ..
drwxr-xr-x. 2 jenkins jenkins   28 May 26 01:08 maven-archiver
drwxr-xr-x. 2 jenkins jenkins    6 May 26 01:08 surefire
drwxr-xr-x. 4 jenkins jenkins   54 May 26 01:08 webapp
-rw-r--r--. 1 jenkins jenkins 2525 May 26 01:08 webapp.war
```



---



##### tomcat-server에서 user는 deployer를 사용한다. (tomcat server에 배포)

- 동일하게 item을 생성한 후 소스 코드 관리, Build를 작성한다. 여기서 추가로 **빌드 후 조치**를 작성한다.

![image](https://user-images.githubusercontent.com/42603919/82851690-8b77ea00-9f3b-11ea-90a5-d1c9e8185907.png)

- jenkins -> tomcat으로 war 파일을 복사하고 싶다. (jenkins입장에서 tomcat ip address(192.168.56.12)를 작성해야한다.)

![image](https://user-images.githubusercontent.com/42603919/82851962-6fc11380-9f3c-11ea-8457-cadd9d638d95.png)

- tomcat에 잘 복사된 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/42603919/82852108-e78f3e00-9f3c-11ea-8849-318081481ff4.png)

```powershell
[vagrant@tomcat-server ~]$ cd /usr/local/tomcat/apache-tomcat-9.0.35
[vagrant@tomcat-server apache-tomcat-9.0.35]$ ls -al
total 128
drwxr-xr-x. 9 root root   220 May 22 07:20 .
drwxr-xr-x. 3 root root    69 May 22 07:20 ..
drwxr-x---. 8 root root   113 May 26 01:34 webapps
```

- 다음 웹 페이지에서 작업을 실시한다.

![image](https://user-images.githubusercontent.com/42603919/82852163-160d1900-9f3d-11ea-8e1d-bbb1b1df6e72.png)

##### 주기적인 build 적용해보기

[crontab](https://jdm.kr/blog/2)

- hello-world/webapp/src/main/webapp/index.jsp를 다음과 같이 수정

```jsp
<h1> Hello, Welcome to Simple DevOps Project !!   </h1>
<h2> Hi there!! Glad to see you here! </h2>
```

- 빌드 유발 설정

![image](https://user-images.githubusercontent.com/42603919/82853675-54a4d280-9f41-11ea-8801-c5d3bf6bbcf4.png)

- 값이 바뀌는 것을 확인

![image](https://user-images.githubusercontent.com/42603919/82853914-f1677000-9f41-11ea-8f1a-86c2cb4eb13e.png)



---





#### [설정]

##### 관리자 암호 변경

<img src="https://user-images.githubusercontent.com/42603919/82634935-04c0c580-9c3a-11ea-8c9a-572dfddfddc2.png" alt="image" style="zoom:67%;" />

##### JAVA 설정

![image](https://user-images.githubusercontent.com/42603919/82636803-6a16b580-9c3e-11ea-8554-45b3021201ef.png)



##### Project 생성

![image](https://user-images.githubusercontent.com/42603919/82636956-be219a00-9c3e-11ea-9d35-a2b9b88ec513.png)

![image](https://user-images.githubusercontent.com/42603919/82637238-4c961b80-9c3f-11ea-9d42-2aa4776a5c96.png)



##### VM에서 접근할 수 있도록 하는 방법

1. 

```powershell
C:\Users\HPE\work\vagrant>vagrant ssh-config jenkins-server
Host jenkins-server
  HostName 127.0.0.1
  User vagrant
  Port 19211
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile C:/Users/HPE/work/vagrant/.vagrant/machines/jenkins-server/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL


C:\Users\HPE\work\vagrant>ssh -p 22 -i C:/Users/HPE/work/vagrant/.vagrant/machines/jenkins-server/virtualbox/private_key vagrant@192.168.56.11
The authenticity of host '192.168.56.11 (192.168.56.11)' can't be established.
ECDSA key fingerprint is SHA256:2MaJs7uadeGFSm8DhBCKkmPnUFcEJYaEtWmjikJrtac.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.56.11' (ECDSA) to the list of known hosts.
Last login: Tue May 26 00:14:04 2020 from 10.0.2.2
[vagrant@jenkins-server ~]$
```



2. 

```powershell
C:\Users\HPE\work\vagrant>ssh -p 19211 -i C:/Users/HPE/work/vagrant/.vagrant/machines/jenkins-server/virtualbox/private_key vagrant@127.0.0.1
The authenticity of host '[127.0.0.1]:19211 ([127.0.0.1]:19211)' can't be established.
ECDSA key fingerprint is SHA256:2MaJs7uadeGFSm8DhBCKkmPnUFcEJYaEtWmjikJrtac.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[127.0.0.1]:19211' (ECDSA) to the list of known hosts.
Last login: Thu May 28 00:28:27 2020 from 192.168.56.1
[vagrant@jenkins-server ~]$
```



##### Docker 실행

```powershell
[vagrant@docker-server ~]$ sudo systemctl start docker
```

```powershell
[vagrant@docker-server ~]$ sudo useradd dockeradmin
[vagrant@docker-server ~]$ sudo passwd dockeradmin
[vagrant@docker-server ~]$ usermod -aG docker dockeradmin
[vagrant@docker-server ~]$ su - dockeradmin
```

```powershell
[dockeradmin@docker-server ~]$ docker pull tomcat:latest
[dockeradmin@docker-server ~]$ docker run -d --rm name tomcat-container --p 8080:8080 tomcat:8
```

- jenkins에 접속하여 다음과 같은 작업 실시

![image](https://user-images.githubusercontent.com/42603919/83089716-f824ed00-a0d1-11ea-98db-8d8777b07f99.png)

<img src="https://user-images.githubusercontent.com/42603919/83089191-afb8ff80-a0d0-11ea-84ee-7f82fdf65275.png" alt="image" style="zoom:67%;" />

##### 빌드 후 조치

**send build artifacts over SSH** 클릭

- Source file : webapp/target/*.war

- Remove prefix : webapp/target

- Excute Command (자동 배포), 밑에 vi Dockerfile부분을 한 다음 docker image를 지우고 다음을 실행

  ```
  cd /home/dockeradmin;docker build -t hello-project .;docker run -d --name hello-container -p 8080:8080 hello-project
  ```



##### docker image를 확인해보면 생성된 것을 확인할 수 있다.

http://localhost:38080/webapp 에 접속해보기

- jenkins에서 작성했던 내용이 보인다.



##### vi Dockerfile(tomcat)

```dockerfile
FROM tomcat:latest
COPY ./webapp.war /usr/local/tomcat/webapp
```

```powershell
[dockeradmin@docker-server ~]$ docker build -t hello-project .
```

```powershell
[dockeradmin@docker-server ~]$ docker run -d --name hello-container -p 8080:8080 hello-project
```

- http://localhost:38080/webapp 에 접속해보기
  - jenkins에서 작성했던 내용이 보인다.





---

##### Maven image로 배포

[Docker Maven image](https://hub.docker.com/_/maven)

```dockerfile
FROM maven:3-openjdk-8-slim
COPY ./webapp.war /usr/src/webapp.jar
ENTRYPOINT ["java", "-jar", "webapp.jar"]
//ENTRYPOINT ["mvn", "spring-boot:run"]
```

---