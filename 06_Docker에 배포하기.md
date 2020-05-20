### Docker에 배포하기

> 1. Docker + AWS EC2
> 2. Docker + Docker Swarm Mode + AWS EC2
> 3. Docker + kubernetes + AWS EC2



#### config-server

##### 다음 파일을 추가

![image](https://user-images.githubusercontent.com/42603919/82408543-c21ab400-9aa6-11ea-9acd-76acd2697dc0.png)

![image](https://user-images.githubusercontent.com/42603919/82408683-132aa800-9aa7-11ea-8414-9ec506ed11e2.png)

1. Dockerfile 작성

```dockerfile
# Dockerfile

FROM openjdk:8-jdk-alpine
VOLUME /tmp
COPY apiEncryptionKey.jks apiEncryptionKey.jks
COPY UnlimitedJCEPolicyJDK8/* /usr/lib/jvm/java-1.8-openjdk/jre/lib/security/
COPY target/myapp-config-server-0.0.1-SNAPSHOT.jar ConfigServer.jar
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar", "ConfigServer.jar"]
```

2. maven clean

```
C:\Users\HPE\msa\msa\LAB\myapp-config-server>mvn clean
```

3. docker build

```
C:\Users\HPE\msa\msa\LAB\myapp-config-server>docker build --tag=dlrjsapdlf622/config-server --force-rm=true .
```

4. docker images 확인

```
C:\Users\HPE\msa\msa\LAB\myapp-config-server>docker images

REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
dlrjsapdlf622/config-server             latest              5c26cf8255bb        3 minutes ago       141MB
```

5. docker image push

```
C:\Users\HPE\msa\msa\LAB\myapp-config-server>docker push dlrjsapdlf622/config-server
```

6. docker hub

![image](https://user-images.githubusercontent.com/42603919/82409991-e6c45b00-9aa9-11ea-978e-7d2f96226b1f.png)



