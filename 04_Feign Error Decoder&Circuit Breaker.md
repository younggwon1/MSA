### Feign Error Decoder

> 에러 값을 정해진 상황에 맞게  출력할 수 있도록 하는 것
>
> 상태 코드를 가지고 제어할 수 있는게 중요(모니터링을 통해서 확인)

```java
# MyappApiUsersApplication.java

    @Bean
    public FeignErrorDecoder getFeignErrorDecoder() {
        return new FeignErrorDecoder();
    }
```



```java
# FeignErrorDecoder.java

package com.example.myappapiusers.shared;


import feign.Response;
import feign.codec.ErrorDecoder;
import org.springframework.http.HttpStatus;
import org.springframework.web.server.ResponseStatusException;

public class FeignErrorDecoder implements ErrorDecoder {
    @Override
    public Exception decode(String methodKey, Response response) {
        switch (response.status()) {
            case 400:
                break;
            case 404:
                if(methodKey.contains("getAlbums")) {
                    return new ResponseStatusException(
                            HttpStatus.valueOf(response.status()),
                            "Users albums are not found"
                    );
                }
                break;
            default:
                return new Exception(response.reason());
        }
        return null;
    }
}
```



##### Config에 있는 application.yml 설정 파일을 이용하여 "Users albums are not found" 출력하기

##### exception.albums-not-found, `동적인 메세지를 다룰 때 유용`



##### C:\Users\HPE\work\dev 밑에 있는 application.yml에 다음과 같이 입력한다.

```yaml
albums:
  url: http://albums-ws/users/%s/albums

  exception:
    albums-not-found: User albums are not found
```



##### 다음 코드를 주석처리한다.

```java
# MyappApiUsersApplication.java

// 다음을 주석처리
//    @Bean
//    public FeignErrorDecoder getFeignErrorDecoder() {
//        return new FeignErrorDecoder();
//    }
```



##### Environment 선언

```java
# FeignErrorDecoder.java

Environment env;

@Autowired
public FeignErrorDecoder(Environment env) {
    this.env = env;
}
```



##### 위의 코드를 좀 더 간단히 만든 것이 다음과 같다.

```java
# FeignErrorDecoder.java

@Value("${albums.exception.albums-not-found}")
String errorMessage;
```

- **@Value** : @Component를 선언한 후, yaml 파일에 등록된 값을 읽어오기 위해서 사용



```java
# FeignErrorDecoder.java

package com.example.myappapiusers.shared;


import feign.Response;
import feign.codec.ErrorDecoder;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.core.env.Environment;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ResponseStatusException;



@Component
@RefreshScope //모든 설정상황을 자동으로 바꿔주는 어노테이션
public class FeignErrorDecoder implements ErrorDecoder {

    @Value("${albums.exception.albums-not-found}")
    String errorMessage;

    // 좀 더 간단히 만든것이 위의 코드이다.
//    Environment env;
//
//    @Autowired
//    public FeignErrorDecoder(Environment env) {
//        this.env = env;
//    }

    @Override
    public Exception decode(String methodKey, Response response) {
        switch (response.status()) {
            case 400:
                break;
            case 404:
                if(methodKey.contains("getAlbums")) {
                    return new ResponseStatusException(
                            HttpStatus.valueOf(response.status()),
                            errorMessage
                    );
                }
                break;
            default:
                return new Exception(response.reason());
        }
        return null;
    }
}
```

- **@RefreshScope** : 모든 설정상황을 자동으로 바꿔주는 어노테이션. 설정 변경시 노드 재시작없이 적용 가능하게 한다.
  - C:\Users\HPE\work\dev 밑에 있는 application.yml에 등록된 albums.exception.albums-not-found에 해당되는 값을 변경시키고 **bus-refresh**를 하면 변경된 값이 적용된다.



---



### Hystrix Circuit Breaker & Feign

##### dependency 추가 (User-ws)

```xml
<dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```



##### @EnableCircuitBreaker 추가

```java
@EnableCircuitBreaker
public class MyappApiUsersApplication {
...
}
```



```yaml
# application.yml(User-ws)

feign:
  hystrix:
    enabled: true
```



```java
# AlbumServiceClient.java

package com.example.myappapiusers.client;

import com.example.myappapiusers.model.AlbumResponseModel;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

import java.util.ArrayList;
import java.util.List;

@FeignClient(name = "ALBUMS-WS", fallback = AlbumsFallback.class// 추가한 부분)
public interface AlbumServiceClient {

    @GetMapping("/users/{id}/albums")
    List<AlbumResponseModel> getAlbums(@PathVariable String id);

}

// 추가한 부분
@Component
class AlbumsFallback implements AlbumServiceClient{
    @Override
    public List<AlbumResponseModel> getAlbums(String id) {
        return new ArrayList<>();
    }
}
```

