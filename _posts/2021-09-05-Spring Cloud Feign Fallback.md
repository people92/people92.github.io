---
layout: post
title:  "Spring Cloud Feign fallback"
date:   2021-09-05
excerpt: "Spring Cloud Feign fallback"
tag:
- feign 
comments: true
---


# Spring Cloud Feign fallback

## 목차

1. Hystrix란?
2. Feign fallback 예제
3. Feign fallback 테스트 시 에러 이슈

___

## __개요__
FeignClient 사용시 Timeout이 종종 발생하는 경우가 있었는데   
이러한 케이스 처리를 위해 Feign Fallback에 대해 알아보고 적용해보려고 한다.  

___

## __상세__

### 1. Hystrix란?
Hystrix란?  
netflix에서 만든 라이브러리로 마이크로 서비스 아키텍처에서 분산된 서비스간 통신이 원활하지 않은 경우에 각 서비스가 장애 내성과 지연 내성을 갖게하도록 도와주는 라이브러리다.  

Hystrix란?
spring-cloud의 서비스 중 하나. Circuit Breaker Pattern을 사용. API 서버가 장애 발생 등의 이유로 일정 시간(Time window) 내에 여러번 오류 응답을 주는 경우(timeout, bad gateway 등)  
해당 API 서버로 요청을 보내지 않고 잠시 동안 대체(fallback) method를 실행. 일정 시간이 지나서 다시 뒷단 API 서버를 호출하는 등의, 일련의 작업을 제공해준다.

fallback이란?  
실패에 대한 후처리를 위해 작업이다.  


### 2. Feign fallback 예제

1. spring-cloud-starter-openfeign maven dependency 추가  
``` xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

2. fallback 처리를 위해 Spring cloud feign hystrix를 사용할 것이기 때문에 application.yml에 true로 설정(default false)  
- 테스트를 위해 Timeout 시간을 1ms로 설정하고 Timeout 발생하도록 함.  
``` yml
feign:
  hystrix:
    enabled: true
  client:
    config:
      default:
        loggerlevel: FULL
        connectTimeout: 1
        readTimeout: 1
```

3. FeignClient interface를 생성  

``` java
@FeignClient(name = "kakao-open-api",
        url = "https://dapi.kakao.com",
        configuration = KakaoFeignConfiguration.class)
public interface KakaoOpenApiClient {

    @RequestMapping(method = RequestMethod.GET, value = "/v2/search/web")
    ResKakaoApi searchDaumWeb(@RequestParam(name = "query") String query);
}

```

4. Fallback 처리를 위해 생성한 FeignClient를 implements하여 Fallback class 생성하고 에러 발생 시 리턴 할 값 설정  
- Fallback 처리 시 빈 리스트가 리턴되도록 생성  

``` java
package com.study.springcloud;

import org.springframework.stereotype.Component;

@Component
public class KakaoOpenApiClientFallback implements KakaoOpenApiClient {

    @Override
    public ResKakaoApi searchDaumWeb(String query) {
        return ResKakaoApi.EMPTY;
    }
}
```

5. FeignClient의 fallback 속성 값에 생성한 Fallback 클래스 설정  

``` java
@FeignClient(name = "kakao-open-api",
        url = "https://dapi.kakao.com",
        configuration = KakaoFeignConfiguration.class,
        fallback = KakaoOpenApiClientFallback.class)
public interface KakaoOpenApiClient {

    @RequestMapping(method = RequestMethod.GET, value = "/v2/search/web")
    ResKakaoApi searchDaumWeb(@RequestParam(name = "query") String query);
}
```
6. Timeout 에러 발생 시 빈 리스트가 리턴되는 것을 확인  
<img src = "https://user-images.githubusercontent.com/28687900/132124652-378c9819-1c08-45e0-a6f7-d37802b5a02d.png">  

7. Fallback 설정을 사용 시 에러메세지는 확인할수 없다.  
따라서 Feign Hystrix에 있는 FallbackFactory를 직접 구현하여 관련 에러 메시지를 확인 할 수 있다.

``` java
package com.study.springcloud;

import feign.hystrix.FallbackFactory;
import lombok.extern.slf4j.Slf4j;

import org.springframework.stereotype.Component;

@Slf4j
@Component
public class KakaoOpenApiClientFallbackFactory implements FallbackFactory<KakaoOpenApiClient> {

    @Override
    public KakaoOpenApiClient create(Throwable cause) {
        return new KakaoOpenApiClient() {
            @Override
            public ResKakaoApi searchDaumWeb(String query) {
                log.error("Feign Client Error : " + cause.getMessage());
                return ResKakaoApi.EMPTY;
            }
        };
    }
}

```

8. FeignClient fallbackFactory 속성에 생성한 클래스 설정
``` java
@FeignClient(name = "kakao-open-api",
        url = "https://dapi.kakao.com",
        configuration = KakaoFeignConfiguration.class,
        //fallback = KakaoOpenApiClientFallback.class
        fallbackFactory = KakaoOpenApiClientFallbackFactory.class
        )
public interface KakaoOpenApiClient {

    @RequestMapping(method = RequestMethod.GET, value = "/v2/search/web")
    ResKakaoApi searchDaumWeb(@RequestParam(name = "query") String query);
}
```

```에러메시지 결과```
``` 
2021-09-05 16:50:04.817 ERROR 13912 --- [akao-open-api-1] c.s.s.KakaoOpenApiClientFallbackFactory  : Feign Client Error : connect timed out executing GET https://dapi.kakao.com/v2/search/web?query=%EC%93%B1%EB%8B%B7%EC%BB%B4
```


### 3. Feign fallback 테스트 시 에러 이슈
1. Feign fallback이 작동하지 않음
- Timeout 발생해도 fallback 처리가 되지 않고 에러 발생  

[해결방법] : Spring boot와 Spring cloud 버전을 바꾸니 정상 작동(원인 모름)  
- 기존 Spring boot version : 2.4.5, spring cloud version : 2020.0.0  
- 변경 Spring boot version 2.3.3.RELEASE, spring cloud version : Hoxton.SR11

2. Fallback은 정상작동 하는데 FallbackFactory를 구현하면 아래 메세지와 같은 에러 발생  

```
Incompatible fallback instance. 
Fallback/fallbackFactory of type class com.study.springcloud.KakaoOpenApiClientFallbackFactory is not assignable to interface com.study.springcloud.KakaoOpenApiClient for feign client kakao-open-api
```

[해결방법] : import를 다른 FallbackFactory를 함.  
- 기존 : import org.springframework.cloud.openfeign.FallbackFactory
- 변경 : import feign.hystrix.FallbackFactory


___


## __참고__
https://cloud.spring.io/spring-cloud-openfeign/reference/html/  
https://brunch.co.kr/@springboot/202
