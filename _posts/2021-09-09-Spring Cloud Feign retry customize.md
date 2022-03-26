---
layout: post
title:  "Spring Cloud Feign retry customize"
date:   2021-09-09
excerpt: "Spring Cloud Feign retry customize"
tag:
- feign 
comments: false
---

# Spring Cloud Feign retry customize

기본적으로 Spring Cloud Feign retry 대상은 IO Exception이다.  

IO Exception이란  
입출력 동작 실패 또는 인터럽트 시 발생하는 Exception

ErrorDecoder 인터페이스를 직접 구현하여 에러 코드별로 처리 할수 있다.  
retry 대상이 아닌 에러 코드를 retry 처리 하려면 아래와 같이 해당 코드에 RetryableException 예외를 발생시켜주면 된다.  
+ yml 설정에 feign.client.config.feignName.errorDecoder에 ErrorDecoder 인터페이스를 직접 구현한 클래스 경로를 추가해준다. 


``` java
package com.study.springcloud;

import feign.Response;
import feign.RetryableException;
import feign.codec.ErrorDecoder;

public class KakaoFeignClientErrorDecoder implements ErrorDecoder {
    private final ErrorDecoder defaultErrorDecoder = new Default();

    @Override
    public Exception decode(String s, Response response) {
        System.out.println("Error decode Test");

        if (404 == response.status()) {
            return new RetryableException(404, response.reason(), response.request().httpMethod(), null, response.request());
        }

        return defaultErrorDecoder.decode(s, response);
    }
}
```

``` yml
feign:
  hystrix:
    enabled: true
  client:
    config:
      default:
        connectTimeout: 10000
        readTimeout: 10000
        loggerlevel: BASIC
      kakao-open-api: # FeignClient에서 name 설정값으로 준 값
        errorDecoder: com.study.springcloud.KakaoFeignClientErrorDecoder
        connectTimeout: 10000
        readTimeout: 10000
        loggerLevel: BASIC
```


404 에러 코드로 테스트를 해보겠다.  
기본적으로 404 에러는 IO Exception이 아니여서 retry 대상이 아니다. 따라서 retry 처리가 되지 않는다.

```
2021-09-09 10:45:36.646 DEBUG 7244 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> GET https://dapi.kakao.com/v2/search/web2?query=SSG HTTP/1.1
2021-09-09 10:45:36.867 DEBUG 7244 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] <--- HTTP/1.1 404 Not Found (218ms)
2021-09-09 10:45:36.874 ERROR 7244 --- [akao-open-api-1] c.s.s.KakaoOpenApiClientFallbackFactory  : Feign Client Error : [404 Not Found] during [GET] to [https://dapi.kakao.com/v2/search/web2?query=SSG] [KakaoOpenApiClient#searchDaumWeb(String)]: [{"errorType":"ResourceNotFound","message":"'GET /search-open/v2/web2' is not matched"}]
```


하지만, 위의 예시대로 404 에러코드일때 강제로 RetryableException 예외를 발생시키면  
retry 처리가 되는 것을 확인 할 수 있다.

```
2021-09-09 10:47:13.867 DEBUG 10432 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> GET https://dapi.kakao.com/v2/search/web2?query=SSG HTTP/1.1
2021-09-09 10:47:14.068 DEBUG 10432 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] <--- HTTP/1.1 404 Not Found (197ms)
Error decode Test
2021-09-09 10:47:14.224 DEBUG 10432 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> RETRYING
2021-09-09 10:47:14.224 DEBUG 10432 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> GET https://dapi.kakao.com/v2/search/web2?query=SSG HTTP/1.1
2021-09-09 10:47:14.234 DEBUG 10432 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] <--- HTTP/1.1 404 Not Found (10ms)
Error decode Test
2021-09-09 10:47:14.461 DEBUG 10432 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> RETRYING
2021-09-09 10:47:14.461 DEBUG 10432 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> GET https://dapi.kakao.com/v2/search/web2?query=SSG HTTP/1.1
2021-09-09 10:47:14.473 DEBUG 10432 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] <--- HTTP/1.1 404 Not Found (12ms)
Error decode Test
2021-09-09 10:47:14.479 ERROR 10432 --- [akao-open-api-1] c.s.s.KakaoOpenApiClientFallbackFactory  : Feign Client Error : Not Found
```


## __참고__
https://medium.com/swlh/how-to-customize-feigns-retry-mechanism-b472202be331  
https://buddhimawijeweera.wordpress.com/2020/01/29/error-decoding-and-retrying-with-feign-clients/  
