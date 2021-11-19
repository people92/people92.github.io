---
layout: post
title:  "Feign Client 사용중 411 Length Required 에러"
date:   2021-11-18
excerpt: "Feign Client 사용중 411 Length Required 에러"
tag:
- markdown 
comments: false
---


# Feign Client 사용중 411 "Length Required" 에러

___


## __문제__
Legacy를 MSA로 변환 중 특정 큐를 읽은 후 삭제하는 로직이 있었는데,  
해당 API는 POST 방식의 HTTP 요청이지만 빈 body로 요청을 하는 경우였다.  
하지만, 그 경우 Feign Client로 호출시 411 "Length Required" 에러가 발생하였다.  
Content-Length: 0으로 헤더를 세팅해도 동일.

___

## __해결방법__

1. body에 빈 스트링을 줘서 처리 함
2. Feign Client에서 제공하는 Feign 종류는 Default, ApacheHttpClient, OkHttpClient 3가지가 있음.  
Default는 기본으로 Empty body여도 뭔가를 생성하는 Feign Client 버그인거 같다.  
따라서 ApacheHttpClient, OkHttpClient 둘중 하나를 사용하면 해결됨.  

Feign client를 HttpClient 나 OkhttpClient로 사용하려면  
아래와 같은 maven을 추가하고 .yml에 enabled: true를 설정하면 바로 동작한다.


``` xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
    <version>10.2.3</version>
</dependency>
```

``` yml
feign:
  httpclient:
    enabled: true
```


``` xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
    <version>10.2.0</version>
</dependency>
```


``` yml
feign:
  okhttp:
    enabled: true
```


___


## __참고__
https://github.com/OpenFeign/feign/issues/1251  
