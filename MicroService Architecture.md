# MicroService Architecture

#### Architecture의 역할

1. 무형의 지식 집합체 (코드)

2. 기능적인 측면보다는 안정적인 운영을 위한 전체적인 구조를 설계 : ex)오케스트라



#### MicroService

1. 중앙 집중 관리를 최소화 

2. 다양한 서비스가 큰 어플리케이션을 구성 -> 다양한 프로그래밍 언어를 구현

3. 데이터 베이스를 다양하게 구성할 수 있다. (다양한 데이터 베이스 ->  Oracle, Mysql ...)

   3.1 로그 처리 -> NoSQL, 화면단 -> 관계형 DB ...



- **RESTful** (데이터를 호출)
- **Small Well Chosen Deployable Units**
- **Cloud Enable**



[**Kafka**](https://engkimbs.tistory.com/691)

[**Service Discovery**](https://futurecreator.github.io/2018/10/18/service-discovery-in-microservices/)



#### Cloud Native

> Cloud Native 환경에서 SaaS나 FaaS 형태로 서비스되는 애플리케이션



# DevOps

> 개발과 운영이 분리되면서 오는 문제점을 해결하기 위해서 개발과 운영을 하나의 조직으로 합쳐서 팀을 운영하는 문화이자 개발 방법론이다.
>
> 엔지니어가, 프로그래밍하고, 빌드하고, 직접 시스템에 배포 및 서비스를 RUN 
>
> 사용자와 끊임 없이 Interaction하면서 서비스를 개선해 나가는 일련의 과정 , 문화

### Agile

1. 가벼운 프로세스

2. **협업 + 피드백** ★

3. 민첩함, 능동적, 자발적, 형식에 구애 받지 않음

4. 반복 점진 개발 + 품질 개선 활동

   4.1 짧은 기간 단위의 반복 절차를 통해 리스크를 줄임

   4.2 개발 주기(계획, 개발, 출시)가 여러 번 반복

5. 고객의 피드백에 민첩하게 반응

6. **Less document-oriented -> code-oriented** ★

   6.1 프로그래밍에 집중하는 유연한 개발 방식

7. eXtreme Programming, Scrum



#### CD + DevOps 기반

- CD (Continuous Delivery)
  - 운영 시스템에 계속해서 Fix나 새로운 기능을 지속적으로 Release를 하는 개념
  - 새로운 FIX나 기능이 추가되면 거의 매일 Release를 하는 개념
- DevOps (Development + Operations)
  - 개발팀 + 운영팀



<img src="https://user-images.githubusercontent.com/42603919/81371270-60fff180-9132-11ea-8433-26eee817ad87.png" alt="image" style="zoom:67%;" />





클라이언트 -> 마이크로 서비스를 이용하기 위해서 API방식을 이용한다. -> endpoint를 resistry 서비스에 등록 (discovery service) 

각각의 도커 컨테이너에 마이크로 서비스를 올린다.



#### CI/CD

Source -> Build -> Test -> Artifact Storage -> Deploy -> Monitor



### [실습]

#### 시나리오

고객이 커피를 주문 -> 주문 서비스(고객에 대한 정보, 주문 프로세스...) -> 커피를 만드는 직원에게 알림 



### Config-Server

- **@SpringBootApplication** : 공통사항으로 필요, @SpringBootApplication으로 되어있는 클래스를 먼저 기동
- **@EnableConfigServer** : config 서버의 역할을 한다는 어노테이션



#### 설정 정보

##### uri: https://github.com/joneconsulting/spring-microservice.git에 있는 정보

```yaml
msaconfig:
  greeting: "hello"
  topic-name: "coffee-topic"
  ipaddress: "192.168.10.1"
  dbtype: "oracle"


-> application.yml
---

spring:
  profiles: local

msaconfig:
  greeting: "Welcome to local server!!!"
  topic-name: "coffee-topic-local"
  ipaddress: "192.168.10.102"
  dbtype: "mysql"
  
-> application-local.yml

...
```

-> **여러개의 yaml 파일을 나눠서 설정을 해도 되고, 아니면 하나의 yaml파일에 모든 설정을 해도 된다.**



##### build.gradle

```
compile('org.springframework.cloud:spring-cloud-config-server') 
//configserve를 사용하기 위해서 필요함.
```



### Eureka-Server

- **@SpringBootApplication** : 공통사항으로 필요, @SpringBootApplication으로 되어있는 클래스를 먼저 기동
- **@EnableEurekaServer** : discovery server(Eureka)가 되기 위해서 사용하는 어노테이션



##### build.gradle

```
compile('org.springframework.cloud:spring-cloud-starter-eureka-server') 
//eurekaserver를 사용하기 위해서 사용함.
```



---



#### 설정(내용) 변경

1. 원래 상태에서 값을 변경 (git에서 변경한다.)

![캡처](https://user-images.githubusercontent.com/42603919/81632032-3b316000-9444-11ea-8a38-e693f99d1419.PNG)

![캡처](https://user-images.githubusercontent.com/42603919/81632372-0245bb00-9445-11ea-96fc-b6139e524195.PNG)

2. 변경된 값을 적용하기 위해서 다음과 같이 실행

![1](https://user-images.githubusercontent.com/42603919/81632028-39679c80-9444-11ea-9932-5640ae4fd678.PNG)



3. 변경된 내용 적용 완료

![2](https://user-images.githubusercontent.com/42603919/81632030-3a98c980-9444-11ea-9b54-07e48bc257f0.PNG)

---



#### 직접 마이크로 서비스 구축해보기

**8010 : Eureka**

**8011 : Zuul**

**8012 : Config**



##### Eureka-server 구축하기

```yaml
# application.yml

server:
  port: 8010

spring:
  application:
    name: DiscoveryService


eureka:
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://localhost:${server.port}/eureka
```



```java
# MyappDiscoveryServiceApplication.java

package com.example.myappdiscoveryservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class MyappDiscoveryServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyappDiscoveryServiceApplication.class, args);
    }

}
```



##### Config-server 구축하기

<img src="https://user-images.githubusercontent.com/42603919/81632987-92383480-9446-11ea-8b60-32e1df506684.png" alt="image" style="zoom:67%;" />

```yaml
# application.yml

server:
  port: 8012

spring:
  application:
    name: ConfigServer

  cloud:
    config:
      server:
        git:
          uri: https://github.com/younggwon1/MyAppConfiguration.git
          username: xxx
          password: xxx
          clone-on-start: true
```



```java
# MyappConfigServerApplication.java

package com.example.myappconfigserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class MyappConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyappConfigServerApplication.class, args);
    }

}
```



##### Application 구축하기

```java
# UsersController.java

package com.example.myappapiusesr.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/users")
public class UsersController {

    //server:
    //  port: 0
    // 랜덤으로 설정된 port번호를 가져오기 위해서 사용.
    @Autowired
    Environment env;

    @GetMapping("/status/check")
    public String status() {

        return String.format("Working on port %s", env.getProperty("local.server.port"));
    }
}
```



```yaml
# application.yml

server:
  port: 0

spring:
  application:
    name: users-ws

devtools:
  restart:
    enable: true
eureka:
  client:
    serviceUrl:
     defaultZone: http://localhost:8010/eureka

gateway:
  ip: 127.0.0.1

token:
  expiration_time: 864000000
  secret: local_secret
```



```yaml
# bootstrap.yml

spring:
  cloud:
    config:
      uri: http://localhost:8012
      name: ConfigServer
```



---



##### apache-maven

[apache-maven](https://maven.apache.org/download.cgi)

- **시스템 환경 설정**

```
M2_HOME : C:\Users\HPE\work\apache-maven-3.6.3

path : %M2_HOME%\bin
```



##### Terminal에서 애플리케이션 실행

```
C:\Users\HPE\msa\msa\LAB\myapp-api-usesr>mvn spring-boot:run
```



### 로드 밸랜서

> 로드밸런서는 서버에 가해지는 부하(=로드)를 분산(=밸런싱)해주는 장치 또는 기술을 통칭한다. 클라이언트와 서버풀(Server Pool, 분산 네트워크를 구성하는 서버들의 그룹) 사이에 위치하며, 한 대의 서버로 부하가 집중되지 않도록 트래픽을 관리해 각각의 서버가 최적의 퍼포먼스를 보일 수 있도록 한다.

- Terminal port : 51951
- IntelliJ port : 35729

마지막에 실행시킨 포트만 살아있다.

![image](https://user-images.githubusercontent.com/42603919/81641581-4b553980-945c-11ea-90f8-0eef714b0f20.png)

![image](https://user-images.githubusercontent.com/42603919/81641674-88b9c700-945c-11ea-85a3-7bf5732cf8a3.png)





#####  이러한 문제를 해결하기 위해 Gateway, Router 역할을 하는 `Zuul`을 사용한다.  



### Zuul

> netflix에서 사용하는 JVM 기반의 라우터이자 로드밸런서이다. 

[참고할 사항](https://taekwang.tistory.com/17)

```java
# MyappZuulGatewayApplication.java

package com.example.myappzuulgateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableZuulProxy
public class MyappZuulGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyappZuulGatewayApplication.class, args);
    }

}
```



```yaml
# application.yml

server:
  port: 8011

spring:
  application:
    name: ZuulServer
eureka:
  client:
    serviceUrl:
     defaultZone: http://localhost:8010/eureka/
```



**그 후, MyappApiUsesrApplication의 application.yml에서 Zuul 서버를 사용할 수 있도록 설정**

```yaml
# application.yml

eureka:
  client:
    serviceUrl:
     defaultZone: http://localhost:8010/eureka
    instance:
      instance-id: ${spring.application.name}:${spring.application.instance_id:${random.value}}
```



다시 IntelliJ와 Terminal에서 서버를 재가동해보면, 다음과 같은 포트가 생성된다.

- Terminal port : 52331
- IntelliJ port : 52482



Zuul서버가 다음을 인식했는지 확인해보자.

**http://localhost:8011/users-ws/users/status/check을 입력하고 새로고침을 눌러보면 다음의 포트(52482, 52331)가 번갈아가면서 화면에 출력된다.**



---



- **IntelliJ**

![image](https://user-images.githubusercontent.com/42603919/81645230-311f5980-9464-11ea-9272-2a0f73c451d1.png)



- **Terminal**

```
C:\Users\HPE\msa\msa\LAB\myapp-api-usesr>mvn spring-boot:run -Dspring-boot.run.arguments="--spring.application.instance_id=young1 --server.port=9001"
```



##### 참고

```
C:\Users\HPE\msa\msa\LAB\myapp-api-usesr>mvn spring-boot:run -Dspring-boot.run.arguments="--spring.applicatio.name=account-ws --spring.application.instance_id=young5 --server.port=0"
```

**Terminal에서도 server.port=0으로 하면 랜덤으로 포트가 설정된다.**



![image](https://user-images.githubusercontent.com/42603919/81645280-46948380-9464-11ea-8c22-3b5e78f4adbc.png)



**http://localhost:8011/users-ws/users/status/check을 입력하고 새로고침을 눌러보면 다음의 포트(랜덤포트, 9000, 9001, 9002)가 번갈아가면서 화면에 출력된다.**



---





