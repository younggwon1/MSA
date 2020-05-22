### Docker에 배포하기

> 1. Docker + AWS EC2
> 2. Docker + Docker Swarm Mode + AWS EC2
> 3. Docker + kubernetes + AWS EC2



#### Intellij에서 서버 실행 없이 docker에 올려 실행할 수 있도록 설정해보자!



#### 임의의 네트워크를 생성하여 모든 컨테이너를 묶을 수 있도록 해보자

- 네트워크 확인하는 명령어

```
C:\Users\HPE>docker network ls
```



1. 네트워크 생성 (이름은 임의로 지정)

```
C:\Users\HPE>docker network create photo-app-network
```



##### ip 확인

```
# 계속 바뀌니 확인해야함!!

docker inspect rabbimq => 172.18.0.2

docker inspect config-server => 172.18.0.3

docker inspect eureka-server => 172.18.0.4

docker inspect zuul-gateway => 172.18.0.5

docker inspect zipkin => 172.18.0.6

docker inspect mysql => 172.18.0.7

docker inspect albums-server => 172.18.0.8
```



#### Rabbitmq

```powershell
docker run -d --name rabbitmq --network photo-app-network -p 15672:15672 -p 5672:5672 -p 15671:15671 -p 5671:5671 -p 4369:4369 --restart=unless-stopped -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:management
```



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

2. maven clean, package

```
C:\Users\HPE\msa\msa\LAB\myapp-config-server>mvn clean
C:\Users\HPE\msa\msa\LAB\myapp-config-server>mvn package
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





![image](https://user-images.githubusercontent.com/42603919/82511447-54bf5f80-9b48-11ea-8176-5a47df74d2a7.png)



##### config-server 실행

```powershell
C:\Users\HPE\msa\msa\LAB\myapp-config-server>docker run -d -p 8012:8012 --network photo-app-network --name config-server -e "spring.rabbitmq.host=172.18.0.2" -e "spring.profiles.active=default" dlrjsapdlf622/config-server
```



##### docker가 가지고 있는 config-server(localhost:8012)

![image](https://user-images.githubusercontent.com/42603919/82511423-43765300-9b48-11ea-85a1-fda70f1c2f9d.png)





---



#### Eureka-Discovery

1. Dockerfile 작성

```dockerfile
# Dockerfile

FROM openjdk:8-jdk-alpine
COPY target/myapp-discovery-service-0.0.1-SNAPSHOT.jar DiscoveryService.jar
ENTRYPOINT ["java", "-jar", "DiscoveryService.jar"]
```

2. maven clean, package

```
C:\Users\HPE\msa\msa\LAB\myapp-discovery-service>mvn clean
C:\Users\HPE\msa\msa\LAB\myapp-discovery-service>mvn package
```

3. docker build

```
C:\Users\HPE\msa\msa\LAB\myapp-discovery-service>docker build --tag=dlrjsapdlf622/eureka-server --force-rm=true .
```

4. docker images 확인

```
C:\Users\HPE\msa\msa\LAB\myapp-discovery-service>docker images

REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
dlrjsapdlf622/eureka-server             latest              5c268g8f9f9        3 minutes ago       141MB
```

5. docker image push

```
C:\Users\HPE\msa\msa\LAB\myapp-discovery-service>docker push dlrjsapdlf622/eureka-server
```

6. docker hub



##### eureka-server 실행

```powershell
C:\Users\HPE\msa\msa\LAB\myapp-config-server>docker run -d -p 8010:8010 --network photo-app-network --name eureka-server -e "spring.cloud.config.url=http://172.18.0.3:8012" dlrjsapdlf622/eureka-server
```



---



#### Zuul-Server

1. Dockerfile 작성

```dockerfile
# Dockerfile

FROM openjdk:8-jdk-alpine
VOLUME /tmp
COPY target/myapp-zuul-gateway-0.0.1-SNAPSHOT.jar ZuulServer.jar
ENTRYPOINT ["java", "-jar", "ZuulServer.jar"]
```

2. maven clean, package

```
C:\Users\HPE\msa\msa\LAB\myapp-zuul-gateway>mvn clean
C:\Users\HPE\msa\msa\LAB\myapp-zuul-gateway>mvn package
```

3. docker build

```
C:\Users\HPE\msa\msa\LAB\myapp-zuul-gateway>docker build --tag=dlrjsapdlf622/zuul-gateway --force-rm=true .
```

4. docker images 확인

```
C:\Users\HPE\msa\msa\LAB\myapp-zuul-gateway>docker images

REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
dlrjsapdlf622/zuul-gateway             latest              g8gf7cf8255bb        3 minutes ago       141MB
```

5. docker image push

```
C:\Users\HPE\msa\msa\LAB\myapp-zuul-gateway>docker push dlrjsapdlf622/zuul-gateway
```

6. docker hub



##### zuul-gateway 실행

```powershell
C:\Users\HPE\msa\msa\LAB\myapp-zuul-gateway>docker run -d -p 8011:8011 --network photo-app-network --name zuul-gateway -e "spring.rabbitmq.host=172.18.0.2" -e "eureka.client.serviceUrl.defaultZone=http://test:test@172.18.0.4:8010/eureka" -e "spring.cloud.config.uri=http://172.18.0.3:8012" dlrjsapdlf622/zuul-gateway
```



---



#### zipkin

```
docker run -d -p 9411:9411 --network photo-app-network openzipkin/zipkin
```



---



#### mysql

```
docker run -d -p 3306:3306 --network photo-app-network --name mysql -e "MYSQL_ROOT_PASSWORD=mysql" -e "MYSQL_DATABASE=photo_app" -e "MYSQL_USER=user" -e "MYSQL_PASSWORD=user" mysql:latest
```



---



#### Albums Service 

1. Dockerfile 작성

```dockerfile
# Dockerfile

FROM openjdk:8-jdk-alpine
VOLUME /tmp
COPY target/myapp-api-albums-0.0.1-SNAPSHOT.jar PhotoAppApiAlbums.jar
ENTRYPOINT ["java", "-jar", "PhotoAppApiAlbums.jar"]
```

2. maven clean, package

```
C:\Users\HPE\MSA-api-albums>mvn clean
C:\Users\HPE\MSA-api-albums>mvn package
```

3. docker build

```
C:\Users\HPE\MSA-api-albums>docker build --tag=dlrjsapdlf622/albums-server --force-rm=true .
```

4. docker images 확인

```
C:\Users\HPE\MSA-api-albums>docker images

REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
dlrjsapdlf622/albums-server             latest              g66j5f8255bb        3 minutes ago       141MB
```

5. docker image push

```
C:\Users\HPE\MSA-api-albums>docker push dlrjsapdlf622/albums-server
```

6. docker hub



##### albums-service 실행

- **--name albums-service** : 인스턴스를 여러개 띄울 수 있기 때문에 이름 설정을 하게되면 충돌이 발생할 수 있다.
- **랜덤 포트**이므로 포트를 지정하지 않는다.

```powershell
C:\Users\HPE\MSA-api-albums>docker run -d --network photo-app-network -e "eureka.client.serviceUrl.defaultZone=http://test:test@172.18.0.4:8010/eureka" dlrjsapdlf622/albums-server
```



![image-20200521140231707](C:\Users\HPE\AppData\Roaming\Typora\typora-user-images\image-20200521140231707.png)



- 포트를 고정시켜 놓는다.(윈도우에서 접근하기위해서 port forwarding을 해준다.)

```powershell
C:\Users\HPE\MSA-api-albums>docker run -d --network photo-app-network -e "eureka.client.serviceUrl.defaultZone=http://test:test@172.18.0.4:8010/eureka" -e "server.port=30000" -p 30000:30000 dlrjsapdlf622/albums-server
```



----



#### Users Service

1. Dockerfile 작성

```dockerfile
# Dockerfile

FROM openjdk:8-jdk-alpine
VOLUME /tmp
COPY target/myapp-api-users-0.1.jar UsersService.jar
ENTRYPOINT ["java", "-jar", "UsersService.jar"]
```

2. maven clean, package

```
C:\Users\HPE\MSA-Project>mvn clean
C:\Users\HPE\MSA-Project>mvn package
```

3. docker build

```
C:\Users\HPE\MSA-Project>docker build --tag=dlrjsapdlf622/users-service --force-rm=true .
```

4. docker images 확인

```
C:\Users\HPE\MSA-Project>docker images

REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
dlrjsapdlf622/users-service             latest              g8gf7cf8f77b        3 minutes ago       141MB
```

5. docker image push

```
C:\Users\HPE\MSA-Project>docker push dlrjsapdlf622/users-service
```

6. docker hub



##### Users Service 실행

```powershell
C:\Users\HPE\MSA-Project>docker run -d --network photo-app-network -e "spring.zipkin.base-url=http://172.18.0.6:9411" -e "spring.rabbitmq.host=172.18.0.2" -e "eureka.client.serviceUrl.defaultZone=http://test:test@172.18.0.4:8010/eureka" -e "spring.cloud.config.uri=http://172.18.0.3:8012" -e "spring.datasource.url=jdbc:mysql://172.18.0.7:3306/photo_app?serverTimezone=Asia/Seoul" -e "spring.datasource.drvier-class-name=com.mysql.cj.jdbc.Driver" -e "spring.datasource.username=user" -e "spring.datasource.password=user" dlrjsapdlf622/users-service
```



- docker를 통해 다음과 같이 서버와 서비스가 올라가 있다.

![image](https://user-images.githubusercontent.com/42603919/82621255-881bf000-9c15-11ea-9ecd-1065eb2a37bc.png)



- 그리고 mysql에 database와 table이 생성된 것을 알 수 있다.

```
C:\Users\HPE>docker exec -it mysql container id bash
root@mysql container id:/# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 19
Server version: 8.0.20 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| photo_app          |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use photo_app
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------------+
| Tables_in_photo_app |
+---------------------+
| hibernate_sequence  |
| users               |
+---------------------+
2 rows in set (0.00 sec)

mysql> desc users;
+--------------------+--------------+------+-----+---------+-------+
| Field              | Type         | Null | Key | Default | Extra |
+--------------------+--------------+------+-----+---------+-------+
| id                 | bigint       | NO   | PRI | NULL    |       |
| email              | varchar(120) | NO   | UNI | NULL    |       |
| encrypted_password | varchar(255) | NO   | UNI | NULL    |       |
| first_name         | varchar(50)  | NO   |     | NULL    |       |
| last_name          | varchar(50)  | NO   |     | NULL    |       |
| user_id            | varchar(255) | NO   | UNI | NULL    |       |
+--------------------+--------------+------+-----+---------+-------+
6 rows in set (0.00 sec)
```



---



#### Postman에서 회원가입 로그인 등등 해보기

...

