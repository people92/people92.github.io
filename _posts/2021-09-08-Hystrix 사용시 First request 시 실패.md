# Hystrix 사용시 First request 시 실패

<img src = "https://user-images.githubusercontent.com/28687900/132426330-0c9800a4-0ab6-456d-8de7-2007dfcdd7af.png">  

### [문제]
Feign과 hystrix 사용시 첫 요청 시 실패하는 경우가 발생한다.  
Feign 문제는 아니고 hystrix 때문에 발생하는 문제이다.  

### [해결]
stackoverflow에서 같은 문제에 대한 답변을 찾아보니  
htstrix default 요청에 대한 timeout 설정이 작아서 그렇다고 한다.  
기본 값이 1초라 요청이 오래 걸리는 경우 반드시 수정이 필요  

```
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds # 요청에 대한 Time Out 설정
```
이 설정은 Hystrix 가 적용된 메서드의 타임아웃을 지정한다.  
따라서 보통 feign read timeout과 connect timeout 지정한 초를 포함하여 여유있게 설정하는 것이 좋다.


``` yml
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 10000 # default : 1000
```

## __참고__

https://stackoverflow.com/questions/39602627/spring-cloud-feign-hytrix-first-call-timeout   
https://github.com/Netflix/Hystrix/wiki/Configuration#executionisolationthreadtimeoutinmilliseconds  
