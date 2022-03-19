---
layout: post
title:  "Linux 환경에서 Jenkins 설치 및 구축"
date:   2022-03-19
excerpt: "Linux 환경에서 Jenkins 설치 및 구축"
tag:
- markdown 
comments: false
---

# Linux 환경에서 Jenkins 설치 및 구축


## 목차
1. Linux 환경 설치(Ubuntu)
2. Jenkins 설치
3. Jenkins 초기 설정


___

## __상세__

### 1. Linux 환경 설치(Ubuntu)  
윈도우에서는 MicroSoft store에서 제공해주는 Ubuntu 앱을 설치하여 쉽게 리눅스 환경을 설치 할 수 있다.

<img src = "https://user-images.githubusercontent.com/28687900/159112210-45243a71-9aef-4638-8744-971b0de836f1.png">


### 2. Jenkins 설치
Jenkins 공식 홈페이지에서 운영체제별 설치 방법에 따라 설치  
&#8251; https://www.jenkins.io/doc/book/installing/linux  

```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

```
$ sudo apt install openjdk-11-jre
```



### 3. Jenkins 초기 설정

``` Jenkins 실행 ```
```
$ sudo service jenkins start 
```
<img src = "https://user-images.githubusercontent.com/28687900/159113794-46f22a40-ebbd-4a12-bb5c-a4506723c66a.png">


``` Jenkins 상태 확인 ```
```
$ sudo service jenkins status
```
<img src = "https://user-images.githubusercontent.com/28687900/159113827-b8f451cf-0ed3-4fab-b591-37667b96cadc.png">


Jenkins 실행 후 최초 접속 시 관리자 비밀번호를 얻어와 입력해야 한다.  

<img src = "https://user-images.githubusercontent.com/28687900/159114308-da571ad2-33ca-4838-b10f-f73dc2b0ac90.png">


``` 최초 접속시 관리자 비밀번호 얻는 곳 ```

```
$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

<img src = "https://user-images.githubusercontent.com/28687900/159113850-041c2a69-a767-4bdb-9de7-c7825a7f97bf.png">

Install suggested plugins 클릭 후 진행하면 자동으로 설치가 진행.  
<img src = "https://user-images.githubusercontent.com/28687900/159114896-4799fe02-0326-4270-b847-72b560182b36.png">

``` 구축 완료 된 Jenkins 메인 화면 ```
<img src = "https://user-images.githubusercontent.com/28687900/159114929-4b32c3ac-99a1-4e12-94a5-9a537346bd53.png">


___


