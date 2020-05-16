# Microservice

### Yaml 파일 통합관리

![image](https://user-images.githubusercontent.com/42603919/82001310-d8370780-9695-11ea-8fa6-5a7187bef30d.png)



![image](https://user-images.githubusercontent.com/42603919/82001355-fc92e400-9695-11ea-9a33-ed196d0bd473.png)



---



### Encryption Types

- **RSA keypair**
  - **Java keytool**을 이용
  - 암호화를 위해서 JCE를 사용한다.
- ***JCE* (*Java Cryptography Extension*)**
  - 자바 플랫폼과 일부 Java 암호화 아키텍쳐(JCA). JCE 는 암호화, 키 생성 및 키 협약 및 메시지 인증 코드 (MAC) 알고리즘을 위한 프레임워크 및 구현을 제공한다.

[JCE 다운로드](https://www.oracle.com/java/technologies/javase-jce8-downloads.html)

![image](https://user-images.githubusercontent.com/42603919/82001864-2d274d80-9697-11ea-8829-d456619d49cd.png)



#### Config Server

1. **symetric key (대칭키 사용)**

   - resource 밑에 bootstrap.yml 추가

     ```yaml
     encrypt:
       key: abcdef1234
     ```

   - 서버 실행 (Eureka Server -> Config Server)

   - Postman

     - encrypt

       ![image](https://user-images.githubusercontent.com/42603919/82002189-20572980-9698-11ea-9d24-6b761b5cdcd7.png)

       

       ![image](https://user-images.githubusercontent.com/42603919/82012001-94ea9200-96b1-11ea-8aa7-88c5303a6fd4.png)

       

     - decrypt

     ![image](https://user-images.githubusercontent.com/42603919/82002267-55fc1280-9698-11ea-956e-766af7950819.png)

     



##### 1. users-ws.yml에 있는 password를 복사하여 encrypt 한다.

<img src="https://user-images.githubusercontent.com/42603919/82002720-7d9faa80-9699-11ea-808e-eddb049c2032.png" alt="image" style="zoom:67%;" />

##### 2. encrypt한 password를 users-ws.yml의 password에 입력하고 저장한다.

##### 3. Config Server를 다시 실행하고 Postman에서 호출해보면 다음과 같이 나온다.

<img src="https://user-images.githubusercontent.com/42603919/82002536-f8b49100-9698-11ea-86da-055731d624d3.png" alt="image" style="zoom:67%;" />

##### 4. 이때  C:\Users\HPE\work\dev 밑에 있는 users-ws.yml에 다음과 같이 입력한다. ({cipher})

```yaml
spring:
    datasource:
        url: jdbc:h2:mem:testdb
        username: sa
        password: '{cipher}ac3c5cce11eda25daa67d4bf39d352af58ed9c1c4a7f7ebc9868ad83f902a8cd'
```

##### 5. Postman에서 확인해보면 원래 입력했던 password가 보인다.

<img src="https://user-images.githubusercontent.com/42603919/82002561-066a1680-9699-11ea-8e5e-ca86bdbbaec4.png" alt="image" style="zoom:67%;" />



2. **Asymetric key(비대칭키 사용)**

   - ```
     C:\Users\HPE\work\dev>keytool -genkeypair -alias apiEncryptionKey -keyalg RSA -keypass "1q2w3e4r" -keystore apiEncryptionKey.jks -storepass "1q2w3e4r"
     ```

   - ![image](https://user-images.githubusercontent.com/42603919/82004061-f5bb9f80-969c-11ea-8f2a-7f73b250e98f.png)

   - Config server bootstrap.yml 수정

     ```yaml
     encrypt:
     #  key: abcdef1234
     
       key-store:
         location: file:///${user.home}/work/dev/apiEncryptionKey.jks
         password: 1q2w3e4r
         alias: apiEncryptionKey
     ```

   - 서버 실행 (Eureka Server -> Config Server)

   - Postman

     - encrypt

     ![image](https://user-images.githubusercontent.com/42603919/82004413-c8bbbc80-969d-11ea-841d-9c49ceddf940.png)

     - decrypt

   ![image](https://user-images.githubusercontent.com/42603919/82004535-1801ed00-969e-11ea-9dc0-edf0861286dd.png)



##### 1. 비대칭키를 복사해 이때  C:\Users\HPE\work\dev 밑에 있는 application.yml에 붙여넣는다.

```yaml
gateway:
  ip: 59.29.224.203
  
token:
  expiration_time: 864000000 #10 days
  secret: AQAy7w1EYKMEOJazy/MrZqmclJNsyAZvkeenAWA3IEYxsrqR0P4UJ++ax1xxLZDr0byKDKjnBz20HIRkB3yjtIDZHljhdWsp....
```

##### 2. Postman에서 결과를 확인해본다.

![image](https://user-images.githubusercontent.com/42603919/82004780-b2623080-969e-11ea-90aa-67338f712f89.png)

##### 3. C:\Users\HPE\work\dev 밑에 있는 application.yml에 다음과 같이 입력한다. ({cipher})

```yaml
gateway:
  ip: 59.29.224.203
  
token:
  expiration_time: 864000000 #10 days
  secret: '{cipher}AQAy7w1EYKMEOJazy/MrZqmclJNsyAZvkeenAWA3IEYxsrqR0P4UJ++ax1xxLZDr0byKDKjnBz20HIRkB3yjtIDZHljhd...'
```

##### 4. 원래 입력했던 모습으로 나온다. 

![image](https://user-images.githubusercontent.com/42603919/82004843-d0c82c00-969e-11ea-85bc-b3f1e03b63b7.png)



---



#### Queuing System 구성

[kafka 다운로드](https://kafka.apache.org/downloads)

![image](https://user-images.githubusercontent.com/42603919/82011801-0413b680-96b1-11ea-9b90-ebd1cda020c2.png)



#### kafka를 이용한 Queuing

##### 실행 순서

- **kafka-server** (cmd1)

```powershell
C:\Users\HPE\work\kafka_2.12-2.3.1>.\bin\windows\kafka-server-start.bat .\config\server.properties
```



- **zookeeper-server** (cmd2)

```powershell
C:\Users\HPE\work\kafka_2.12-2.3.1>.\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties
```



- **kafka-topic** 생성 (cmd3)

```powershell
C:\Users\HPE\work\kafka_2.12-2.3.1>.\bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic msa_20200515
```



- **topic list** 확인 (cmd3)

```powershell
C:\Users\HPE\work\kafka_2.12-2.3.1>.\bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092

msa_20200515
```



- **구독자(소비자)** 등록 (cmd3)
  - 등록되면 커서가 깜빡인다.

```powershell
C:\Users\HPE\work\kafka_2.12-2.3.1>.\bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic msa_20200515
```



- **발행자** 등록 (cmd4)
  - 등록되면 >와 함께 커서가 깜빡인다.

```powershell
C:\Users\HPE\work\kafka_2.12-2.3.1>.\bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic msa_20200515
>
```



##### [console test]

- 발행자 쪽에서 메세지를 보내보면 구독자에 나타난다. (메시지를 통해 통신)

![image](https://user-images.githubusercontent.com/42603919/82013468-5a82f400-96b5-11ea-9ceb-c6327cc244c0.png)



---



### [실습]

#### 실행 순서

1. **Zookeeper, Kafka Server (CMD 창에서)**
2. **Config Server (8888)**
3. **Eureka Server (9091)**
4. **Zuul Server (9090)**
5. **Turbin Server (9999)**
6. **Hystix Dashboard (7070)**
7. **Coffee api (8080(order), 8081(member), 8082(status))**



#### 확인

1. http://localhost:9091 접속

   ![image](https://user-images.githubusercontent.com/42603919/82019755-bf911680-96c2-11ea-9d1f-0d3c5ef5ee94.png)

2. http://localhost:7070/hystrix 접속 -> http://localhost:9999/turbine.stream입력

3. ![image](https://user-images.githubusercontent.com/42603919/82019927-0da61a00-96c3-11ea-9a48-c5314a8949e7.png)


![image](https://user-images.githubusercontent.com/42603919/82110053-54a11780-9776-11ea-8bf1-22e257837a48.png)



##### Member Table 생성

```java
# CoffeeMemberRestController.java
    
@RequestMapping(value = "/createMemberTable", method = RequestMethod.PUT)
public void createMemberTable() {
    iCoffeeMemberMapper.createMemberTable();
}
```

![image](https://user-images.githubusercontent.com/42603919/82108343-d12cf980-9768-11ea-979d-9c3e2e83d150.png)



##### Member data 삽입

```java
# CoffeeMemberRestController.java
    
@RequestMapping(value = "/insertMemberData", method = RequestMethod.PUT)
public void insertMemberData() {
    iCoffeeMemberMapper.insertMemberData();
}
```

![image](https://user-images.githubusercontent.com/42603919/82108351-ddb15200-9768-11ea-9a9b-c4aa675189f3.png)



- http://localhost:8081/console 접속
- member table이 생성되었고 member가 입력되었음을 확인

![image](https://user-images.githubusercontent.com/42603919/82108436-5b755d80-9769-11ea-9437-584352aacb29.png)





##### 주문 처리 상태 확인 테이블 생성

![image](https://user-images.githubusercontent.com/42603919/82108374-f9b4f380-9768-11ea-9d77-8918afc5a688.png)



- http://localhost:8080/console 접속
- 주문 처리 상태 확인 테이블이 생성

![image](https://user-images.githubusercontent.com/42603919/82108525-0259f980-976a-11ea-8da1-3119002d738f.png)





##### coffee 주문

![image](https://user-images.githubusercontent.com/42603919/82108563-30d7d480-976a-11ea-9756-c2bcb05a40c5.png)



- 회원 확인 요청

  ![image](https://user-images.githubusercontent.com/42603919/82108578-4947ef00-976a-11ea-9eb3-07f424d8fa7b.png)

  

  - 주문 정보를 메시지큐에 발행 (Order)

  ![image](https://user-images.githubusercontent.com/42603919/82108602-81e7c880-976a-11ea-9bea-d3d989e469ec.png)

  - 커피 주문 마이크로서비스에서 발행한 커피 주문 내역 메시지를 구독 (Status)

<img src="https://user-images.githubusercontent.com/42603919/82108593-6d0b3500-976a-11ea-9f58-2d45a6687e20.png" alt="image" style="zoom:67%;" />



##### 주문 상태 확인

![image](https://user-images.githubusercontent.com/42603919/82108653-f02c8b00-976a-11ea-8a11-c480f1469470.png)



##### Hystrix Dashboard에서 확인해보기

![image](https://user-images.githubusercontent.com/42603919/82108670-17835800-976b-11ea-9be4-fdd6822c8011.png)



![image](https://user-images.githubusercontent.com/42603919/82108687-28cc6480-976b-11ea-9b1e-0b121ee3b07e.png)



##### Circuit Breaker (fallback)

```java
# CoffeeMemberRestController.java

    @HystrixCommand(fallbackMethod = "fallbackFunction", commandProperties = {
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",
                    value="3000"),
            @HystrixProperty(name="circuitBreaker.errorThresholdPercentage",
                    value="50"),
            @HystrixProperty(name="circuitBreaker.requestVolumeThreshold",
                    value="2"),
            @HystrixProperty(name="circuitBreaker.sleepWindowInMilliseconds",
                    value="5000")
    })
    @RequestMapping(value = "/fallbackTest", method = RequestMethod.GET)
    public String fallbackTest() throws Throwable {
        throw new Throwable("fallbackTest");
    }

    public String fallbackFunction(Throwable t) {
        log.info(t.getMessage());
        return "fallbackFunction()";
    }
```



![image](https://user-images.githubusercontent.com/42603919/82109252-7945c100-976f-11ea-93a0-d205ee070a8b.png)



![image](https://user-images.githubusercontent.com/42603919/82109264-8a8ecd80-976f-11ea-915c-375925bfb0b8.png)



![image](https://user-images.githubusercontent.com/42603919/82109291-a5f9d880-976f-11ea-9f27-dc0012be398a.png)