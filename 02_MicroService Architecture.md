# MicroService 실습

1. **Eureka Server (8010)**

2. **Config Server (8012)**

   - **git repository**

3. **Zuul (token.secret)... (8011)**

   - **configuration server (x)**
   - **bootstrap.yml**

4. **users-ws**

   - **configuration server(o)**

   - **회원가입**
   - **로그인(Token)**
   - **/status/check -> Bearer**



#### 회원가입

```java
# CreateUserRequestModel.java

package com.example.myappapiusesr.model;

import lombok.Data;


import javax.validation.constraints.Email;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

// 입력하는 부분
@Data
public class CreateUserRequestModel {

    @NotNull
    @Size(min=2)
    private String firstName;
    @NotNull
    @Size(min=2)
    private String lastName;
    @NotNull
    @Email
    private String email;
    @NotNull
    @Size(min = 2, max = 16)
    private String password;
}
```



```java
# CreateUserReponseModel.java

package com.example.myappapiusesr.model;

import lombok.Data;

// 출력되는 부분
@Data
public class CreateUserReponseModel {
    private String firstName;
    private String lastName;
    private String email;
    private String userId;
}
```



![회원가입](https://user-images.githubusercontent.com/42603919/81883132-6a280d00-95cf-11ea-8d64-12f01b846998.PNG)



#### 로그인

```java
//로그인을 하게되면 token 정보와 usserId가 반환
res.addHeader("token", token);
res.addHeader("userid", userDetail.getUserId());
```



- **userId, token이 할당.**

![로그인](https://user-images.githubusercontent.com/42603919/81883136-6b593a00-95cf-11ea-98a3-330f178b7577.PNG)



![로그인1](https://user-images.githubusercontent.com/42603919/81883138-6b593a00-95cf-11ea-9c25-f825c292a3ae.PNG)



#### Token (Bearer)

- token 정보가 넘어가면 값이 출력되고, token 정보가 넘어가지 않으면 값이 출력되지 않는다.

![캡처](https://user-images.githubusercontent.com/42603919/81894576-0f041380-95eb-11ea-99e5-c16e151ee07e.PNG)



---



### Spring Cloud Bus

> `Config Client` 인스턴스마다 `/actuator/refresh`를 호출해줘야하는 단점은 `Spring Cloud Bus`를 통해서 하나의 `메시지 브로커(여기서는 RabbitMQ)`에 모든 인스턴스들을 연결해서 해결할 수가 있다.

[참고사항1](http://blog.eomdev.com/springcloud/2019/04/01/Spring-Cloud-Bus.html)

[참고사항2](https://erea.tistory.com/20)

- **ConfigServer** Dependency

  - Spring Cloud Bus를 사용하기 위해서 spring-cloud-starter-bus-amqp를 추가

  ```xml
  <!-- actuator, ampq-->
  <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  
  <dependency>
       <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
  </dependency>
  ```

- **ZuulServer, User-ws** Dependency

  - Spring Cloud Bus를 사용하기 위해서 spring-cloud-starter-bus-amqp를 추가

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-bus-amqp</artifactId>
  </dependency>
  ```



#### Rabbimq

[rabbitmq 다운로드](https://www.rabbitmq.com/download.html)

- **Docker image**로 다운로드 및 실행

  - **Command** 입력

    - 위치 : git에 올라가 있는 git config(application.yml)가 있는 위치
    - (C:\Users\HPE\work\git\MyAppConfiguration)

    ```
    docker run -d --name rabbitmq -p 5672:5672 -p 9090:15672 --restart=unless-stopped -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:management
    ```

![image](https://user-images.githubusercontent.com/42603919/81901090-f13dab00-95f8-11ea-96c0-4046c6eb9fbc.png)



- **Rabbimq yaml 파일 설정** (Config Server, ZuulServer, User-ws)
  - `RabbitMQ` 정보 등록

```yaml
# application.yml

spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: admin
    password: admin
```

- **localhost:9090**으로 접속하여 username: admin, password: admin을 입력하여 접속



#### 작동 순서

1. **Rabbitmq Server 실행** (Docker image로 실행)
2. **Service Discovery Server 실행**
3. **Zuul Server 실행**
4. **Config Server 실행**
5. **User-ws 실행**



#### bus-refresh

> 수정된 값을 적용하기 위해 사용

**ConfigServer**  **yaml 파일**

- `actuator`의 `/bus-refresh`를 사용가능하도록 노출시켜 주기 위해서 `management.endpoints.web.exposure.include`에 등록해준다.

```yaml
# apllication.yml

management:
  endpoints:
    web:
      exposure:
        include: bus-refresh
```

##### bus-refresh 적용해보기

1. git에서 값을 바꾼다. (or 변경된 파일을 git push한다.)
2. Postman에서 다음 작업 실행
3. http://localhost:8012/actuator/bus-refresh (값이 변경되도록 하기위해서 작업을 실행)
4. 다시 로그인 (Send -> 새로운 토큰 복사)
   - http://localhost:8011/users-ws/users/login
5. 바뀐 값 확인 (복사한 토큰을 붙여넣기 -> Send)
   - http://localhost:8011/users-ws/users/status/check