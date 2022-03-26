---
layout: post
title:  "Jenkins gradle 빌드 및 에러 해결 방법"
date:   2022-03-26
excerpt: "Jenkins gradle 빌드 및 에러 해결 방법"
tag:
- markdown 
- jenkins
comments: false
---


# Jenkins gradle 빌드 및 에러 해결 방법


## 목차
1. Jenkins gradle 빌드
2. build 시 에러 해결 방법
___


## __상세__

### 1. Jenkins gradle 빌드

일단, Jenkins 서버에 gradle을 설치.(리눅스 환경에 젠킨스 서버를 구성하였음.)

```
sudo apt install gradle
```

Jenkins에 Gradle 설정
```
Jenkins 관리 >> Global Tool Configuration >> Gradle >> Gradle 버전 선택 및 Install automatically 체크
```

<img src = "https://user-images.githubusercontent.com/28687900/159509511-13b25487-c88c-4bc0-a5d9-c699a068c975.png">

<img src = "https://user-images.githubusercontent.com/28687900/159876219-173b4545-c31b-499a-8c02-52c7c43b94aa.png">

<img src = "https://user-images.githubusercontent.com/28687900/159875492-53361e88-346b-4bcd-ae95-bcc3c1de8973.png">

Jenkins 빌드 Item 생성

```
새로운 Item 생성 >> Build >> gradle version(위에 설정해돈 Gradle name 작성) 
>> Tasks에 clean, build 입력 >> Build File에 build.gradle 입력
```
<img src = "https://user-images.githubusercontent.com/28687900/159876414-3d0e49d1-06c8-4d43-a84c-dad1b7a0e880.png">

<img src = "https://user-images.githubusercontent.com/28687900/159876470-5bb5b869-4b34-459c-96da-88ca207b15ef.png">


### 2. build 시 에러 해결 방법


1. Spring Boot plugin requires Gradle 6.8.x, 6.9.x, or 7.x. The current version is Gradle 4.4.1  
    : Spring boot 버전과 gradle 버전이 안 맞아서 에러 발생  

<img src = "https://user-images.githubusercontent.com/28687900/159874131-7a7c6031-5b9b-40cc-ae40-b7d293e1c990.png">

해결방법 : gradle 버전에 맞게 재설치
```
wget https://services.gradle.org/distributions/gradle-6.8.1-bin.zip -P /tmp
sudo unzip -d /opt/gradle /tmp/gradle-*.zip
```

```
sudo vi /etc/profile.d/gradle.sh

---gradle 버전 수정
export GRADLE_HOME=/opt/gradle/gradle-6.8.1
export PATH=${GRADLE_HOME}/bin:${PATH}

source /etc/profile.d/gradle.sh
```

```
gradle -v
```
<img src = "https://user-images.githubusercontent.com/28687900/160233619-9699211f-1737-4fa9-a985-e1bd06e1a735.png">


2. DataSourceProperties 관련 에러  
    : Spring batch는 필수적으로 사용하는 메타 테이블들이 있다. 그래서 DB 설정이 필수적

```
SpringBatchJenkinsApplicationTests > contextLoads() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:132
        Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:800
            Caused by: org.springframework.beans.factory.BeanCreationException at ConstructorResolver.java:658
                Caused by: org.springframework.beans.BeanInstantiationException at SimpleInstantiationStrategy.java:185
                    Caused by: org.springframework.boot.autoconfigure.jdbc.DataSourceProperties$DataSourceBeanCreationException at DataSourceProperties.java:252
```

해결방법 : DB 정보 추가
```
spring:
  datasource:
    driver-class-name: org.postgresql.Driver
    url: jdbc:postgresql://localhost:5432/postgres
    username: postgres
    password: 0000
```



___

