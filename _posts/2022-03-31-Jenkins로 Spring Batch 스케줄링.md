---
layout: post
title:  "Jenkins로 Spring Batch 스케줄링"
date:   2022-03-31
excerpt: "Jenkins로 Spring Batch 스케줄링"
tag:
- jenkins 
comments: false
---


# Jenkins로 Spring Batch 스케줄링


## 목차
1. Jenkins를 이용한 스케줄링
2. Spring batch 멱등성
3. Spring batch 실행



___




## __상세__

### 1. Jenkins를 이용한 스케줄링
jenkins은 스케줄링 주기를 설정할때 cron expression을 사용한다.  
10분 마다 실행하는 cron 표현식을 작성하여 테스트 

```
# 10분 마다 실행
* /10 * * * *
```
Build periodically 체크 후 cron 표현식 작성  
<img src = "https://user-images.githubusercontent.com/28687900/160622802-c5689e51-dbb1-43b9-a955-0bca09652625.png">

jenkin gradle 빌드가 되는 경로는  /var/lib/jenkins/workspace/프로젝트명 경로에 빌드가 된다.  
gradle jar가 위치하는곳은 /build/libs 안에 jar파일이 생성되어 있다.

스케줄링 시킬 Shell Command 작성  
jar파일 실행 명령어 작성 + date 변수 넘김
<img src = "https://user-images.githubusercontent.com/28687900/160855712-6f84aac7-c2fa-4a4b-91c8-a7747e9920b6.png">


10분마다 배치 실행되는것 확인  
<img src = "https://user-images.githubusercontent.com/28687900/160865835-1da42a0a-d005-4479-95d1-012ce45aa9c9.png">



### 2. Spring batch 멱등성

Spring Batch는 동일한 파라미터가 입력 시 실행되지 않기 때문에 멱등성을 유지한다.  
따라서, 멱등성을 유지하기 위해 파라미터를 매번 다르게 실행시켜야 함.  
&#8251; 멱등성 : 연산을 여러번 적용하더라도 결과가 달라지지 않는 성질


jenkins plugin 중 ```Data Parameter Plugin```을 사용하여 현재날짜시간을 파라미터로 넘길수 있음.  
ex) LocalData.now(), LocalDate.now().plusDays(1), LocalDateTime.now()


<img src = "https://user-images.githubusercontent.com/28687900/160855965-cfab05d8-79c0-4aeb-9eb7-545a28c5914f.png">


```
이 빌드는 매개변수가 있습니다. 체크 >> Date Parameter >> 변수 Name > Format >> Defaul Values에 제공해주는 함수 작성
```
<img src = "https://user-images.githubusercontent.com/28687900/160855850-14968b76-729c-4460-bfb4-79c4e7f343d4.png">


### 3. Spring batch 실행

```
※ 같은 파라미터로 스프링 배치가 실행되면 아래와 같은 메세지가 출력된다.
Step already complete or not restartable, so no action to execute
```

``` 정상 수행 된 스프링 배치 실행 콘솔 로그 ```
```
Started by timer
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/spring-batch-job
[spring-batch-job] $ /bin/sh -xe /tmp/jenkins1888436300296350493.sh
+ echo start
start
+ echo 20220330233000
20220330233000
+ java -jar /var/lib/jenkins/workspace/test2/build/libs/spring-batch-jenkins-0.0.1-SNAPSHOT.jar date=20220330233000

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.6.4)

2022-03-30 23:30:00.744  INFO 248 --- [           main] c.e.s.SpringBatchJenkinsApplication      : Starting SpringBatchJenkinsApplication using Java 11.0.14 on DESKTOP-TKC9PP5 with PID 248 (/var/lib/jenkins/workspace/test2/build/libs/spring-batch-jenkins-0.0.1-SNAPSHOT.jar started by jenkins in /var/lib/jenkins/workspace/spring-batch-job)
2022-03-30 23:30:00.746  INFO 248 --- [           main] c.e.s.SpringBatchJenkinsApplication      : No active profile set, falling back to 1 default profile: "default"
2022-03-30 23:30:01.100  INFO 248 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
2022-03-30 23:30:01.110  INFO 248 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 3 ms. Found 0 JPA repository interfaces.
2022-03-30 23:30:01.330  INFO 248 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2022-03-30 23:30:01.462  INFO 248 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2022-03-30 23:30:01.558  INFO 248 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
2022-03-30 23:30:01.594  INFO 248 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 5.6.5.Final
2022-03-30 23:30:01.711  INFO 248 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.2.Final}
2022-03-30 23:30:01.803  INFO 248 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.PostgreSQL10Dialect
2022-03-30 23:30:01.911  INFO 248 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
2022-03-30 23:30:01.917  INFO 248 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2022-03-30 23:30:01.977  WARN 248 --- [           main] o.s.b.a.batch.JpaBatchConfigurer         : JPA does not support custom isolation levels, so locks may not be taken when launching Jobs
2022-03-30 23:30:01.980  INFO 248 --- [           main] o.s.b.c.r.s.JobRepositoryFactoryBean     : No database type set, using meta data indicating: POSTGRES
2022-03-30 23:30:02.095  INFO 248 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : No TaskExecutor has been set, defaulting to synchronous executor.
2022-03-30 23:30:02.159  INFO 248 --- [           main] c.e.s.SpringBatchJenkinsApplication      : Started SpringBatchJenkinsApplication in 1.733 seconds (JVM running for 2.056)
2022-03-30 23:30:02.171  INFO 248 --- [           main] o.s.b.a.b.JobLauncherApplicationRunner   : Running default command line with: [date=20220330233000]
2022-03-30 23:30:02.245  INFO 248 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=basicJob]] launched with the following parameters: [{date=20220330233000}]
2022-03-30 23:30:02.290  INFO 248 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [sampleStep2]
2022-03-30 23:30:02.295  INFO 248 --- [           main] c.e.springbatchjenkins.SampleJobConfig   : job parameter : 20220330233000
2022-03-30 23:30:02.295  INFO 248 --- [           main] c.e.springbatchjenkins.SampleJobConfig   : sample step Test2
2022-03-30 23:30:02.300  INFO 248 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [sampleStep2] executed in 9ms
2022-03-30 23:30:02.303  INFO 248 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=basicJob]] completed with the following parameters: [{date=20220330233000}] and the following status: [COMPLETED] in 44ms
2022-03-30 23:30:02.307  INFO 248 --- [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2022-03-30 23:30:02.309  INFO 248 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2022-03-30 23:30:02.313  INFO 248 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
Finished: SUCCESS
```


___
