---
layout: post
title:  "Spring Cloud Feign Response 객체"
date:   2021-11-21
excerpt: "Spring Cloud Feign Response 객체"
tag:
- markdown 
comments: false
---


# Spring Cloud Feign Response 객체

___


## __개요__
Legacy를 MSA로 전환 중인데 모든 API를 Feign을 이용하여 전환하려고 개발중이다.  
Response Header를 가져와야 하는 경우도 있고, Http Status를 가져와서 처리해야 하는 경우도 있다.  
Feign은 Response 객체를 제공하고 있어 이 객체로 리턴 타입을 설정하면 가져 올 수 있는데  
Response에 다른 값들도 존재하고 있어 한번 살펴보려고 한다.  

___

## __상세__


``` Feign.Response 객체```
``` java
public final class Response implements Closeable {
    private final int status;
    private final String reason;
    private final Map<String, Collection<String>> headers;
    private final Response.Body body;
    private final Request request;

    ...
}
```

Feign에서 제공해주는 Response 데이터
- status : HTTP Status Code  
- reason : HTTP 결과 사유(정상일때는 OK 리턴)  
- headers : HTTP Header를 Map 형식으로 가지고 있음  
- body : Response Body 값  
- request : 요청에 대한 정보들  


실제 Body에 있는 본문 내용만 가져오려면 Feign에서 제공해주는 StringDecoder를 이용하면 쉽게 가져올 수 있다.

``` StringDecoder 사용 예시 ```

``` java
    private StringDecoder stringDecoder = new StringDecoder();

    public void findResponse(String query) throws IOException {
        Response response = kakaoOpenApiClient.searchResponse(query);

        String body = stringDecoder.decode(response, String.class).toString();

        ...
    }

```


StringDecoder Class를 보면 Body만 가져와 리턴해주고 있다.
``` Feign.StringDecoder 객체 ```
``` java
public class StringDecoder implements Decoder {
    public StringDecoder() {
    }

    public Object decode(Response response, Type type) throws IOException {
        Body body = response.body();
        if (body == null) {
            return null;
        } else if (String.class.equals(type)) {
            return Util.toString(body.asReader(Util.UTF_8));
        } else {
            throw new DecodeException(response.status(), String.format("%s is not a type supported by this decoder.", type), response.request());
        }
    }
}

```


```결과```
```
2021-11-21 20:43:03.110  INFO 16920 --- [           main] com.study.springcloud.FeignService       : status :200
2021-11-21 20:43:03.114  INFO 16920 --- [           main] com.study.springcloud.FeignService       : headers : {"access-control-allow-headers":["Authorization, KA, Origin, X-Requested-With, Content-Type, Accept"],"access-control-allow-methods":["GET, OPTIONS"],"access-control-allow-origin":["*"],"connection":["keep-alive"],"content-type":["application/json;charset\u003dUTF-8"],"date":["Sun, 21 Nov 2021 11:43:00 GMT"],"server":["nginx"],"transfer-encoding":["chunked"],"vary":["Accept-Encoding"],"x-request-id":["2dec7450-4ac0-11ec-8a91-cf8277894b23"]}
2021-11-21 20:43:03.115  INFO 16920 --- [           main] com.study.springcloud.FeignService       : body : {"documents":[{"contents":"with lowdown and Markdown.pl Make a static site with find(1), grep(1), and lowdown or Markdown.pl \u003cb\u003essg\u003c/b\u003e is a static site generator written in shell. Optionally it converts Markdown files to HTML with...","datetime":"2018-03-28T23:24:37.000+09:00","title":"Usage","url":"https://www.romanzolotarev.com/ssg.html"},...
2021-11-21 20:43:03.116  INFO 16920 --- [           main] com.study.springcloud.FeignService       : reason : OK
2021-11-21 20:43:03.116  INFO 16920 --- [           main] com.study.springcloud.FeignService       : request url :https://dapi.kakao.com/v2/search/web?query=SSG
2021-11-21 20:43:03.118  INFO 16920 --- [           main] com.study.springcloud.FeignService       : request headers : {"Authorization":["KakaoAK 25ca90303af8723009e137e52babb4be"]}
2021-11-21 20:43:03.118  INFO 16920 --- [           main] com.study.springcloud.FeignService       : request httpMethod : GET

```

___


## __참고__
