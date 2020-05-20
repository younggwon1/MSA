### Monitoring

**zipkin** : 호출관계를 보여준다. Zipkin과 연동하기 위해서는 **Sleuth**라는 라이브러리를 사용하면 된다.



#### zipkin 실행

[zipkin 다운로드](https://zipkin.io/pages/quickstart.html)



![image](https://user-images.githubusercontent.com/42603919/82394217-67239580-9a83-11ea-9f17-c226e482edb1.png)



```powershell
C:\Users\HPE\work>java -jar zipkin-server-2.21.1-exec.jar
```



```xml
# User service, pom.xml

<!-- zipkin 추가 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```



```yaml
# User service, application.yml

spirng:
  zipkin:
    base-url: http://localhost:9411
    sender:
      type: web
  sleuth:
    sampler:
      probability: 1 #로그를 수집할 수 있는 확률
```



---



### Eureka Server 보안

> 현재 locallhost:8010을 입력하면 dashboard에 들어가진다. 따라서 이를 막기위해 보안처리를 해본다.

```xml
# Eureka, pom.xml

<!-- Security 추가 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```



```yaml
# Eureka, application.yml

spring:
  application:
    name: DiscoveryService
  security:
    user:
      name: test
      password: test
```



```java
# Eureka, WebSecurity.java

package com.example.myappdiscoveryservice.security;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class WebSecurity extends WebSecurityConfigurerAdapter {


    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .httpBasic();
    }
}

```



![image](https://user-images.githubusercontent.com/42603919/82396056-f3d05280-9a87-11ea-85e5-f2406ae277de.png)



##### Zuul Server, User service를 실행하면 Eureka Server에 접근할 수 없다고 콘솔에 찍힌다. 따라서 다음과 같은 설정을 하고 다시 실행하면 정상적으로 실행이 된다.(아이디와 패스워드를 입력)

```yaml
# Zuul,User application.yml

eureka:
  client:
    serviceUrl:
      defaultZone: http://test:test@localhost:8010/eureka
```



##### 이번에는 Configuration에 정보를 옮겨 적용해보자.

1. ##### discovery service

   ```yaml
   # Eureka, application.yml
   
   spring:
     application:
       name: DiscoveryService
   #  security:
   #    user:
   #      name: test
   #      password: test
   ```

2. move to the **configuration server**

   #####  C:\Users\HPE\work\dev 밑에 있는 discoveryservice.yml에 다음과 같이 입력한다.

   ```yaml
   # discoveryservice.yml
   
   spring:
       security:
         user:
           name: test
           password: test
   ```

3. Eureka에서 bootstrap.yml을 추가하고 다음과 같이 입력

   ```yaml
   spring:
     cloud:
       config:
         uri: http://localhost:8012
         name: ConfigServer
   ```

4. Eureka에서 pom.xml에 다음과 같은 dependency를 추가

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-config</artifactId>
   </dependency>
   ```

5. **encryption** (user, password)

   - 그전에 배웠던 방법으로 진행



---



