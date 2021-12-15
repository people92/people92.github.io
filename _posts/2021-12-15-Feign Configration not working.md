---
layout: post
title:  "Feign Configration not working"
date:   2021-12-15
excerpt: "Feign Configration not working"
tag:
- markdown 
comments: false
---

# Feign Configration 반영 안되는 현상 원인 및 해결


## __개요__
Feign Header 넣는 Config를 따로 만들어  
Feign Configuration에 적용하엿으나 반영이 안되는 현상이 발생.

___

## __상세__
원인 : 같은 FeignClient name이지만 다른 Configuration일때 반영 안됨.  

해결 : FeignClient name 변경하니 정상 작동.

-> KakaoOpenApiClient에서 KakaoFeignConfiguration가 반영되어야 하지만 KakaoTestConfig가 반영됨.  
   Feign name 별로 Configuration이 설정됨 name 같은 걸로 안하게 주의해야 할듯.

``` java
@FeignClient(name = "kakao",
        url = "https://dapi.kakao.com",
        configuration = KakaoFeignConfiguration.class,
        fallbackFactory = KakaoOpenApiClientFallbackFactory.class
        )
public interface KakaoOpenApiClient {
    ...
}
```

``` java
@FeignClient(name = "kakao",
        url = "https://www.naver.com",
        configuration = KakaoTestConfig.class
)
public interface KakaoOpenApiClient2 {
    ...
}
```



___

