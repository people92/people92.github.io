---
layout: post
title:  "Jenkins github 연동 및 브랜치 별 빌드"
date:   2022-03-23
excerpt: "Jenkins github 연동 및 브랜치 별 빌드"
tag:
- markdown 
comments: false
---


# Jenkins github 연동 및 브랜치 별 빌드


## 목차
1. Jenkins github 연동
2. Jenkins 브랜치 별 빌드(매개변수 이용)

___


## __상세__

### 1. Jenkins github 연동

```
새로운 Item >> item name 작성 >> Freestyle project 생성 
```

<img src = "https://user-images.githubusercontent.com/28687900/159509511-13b25487-c88c-4bc0-a5d9-c699a068c975.png">

<img src = "https://user-images.githubusercontent.com/28687900/159509743-1deac597-c700-4857-b10d-dce204518b9e.png">


```
소스코드 관리 >> git repository URL(git 주소 작성) >> branch name 작성
```
<img src = "https://user-images.githubusercontent.com/28687900/159511882-e61b3f3b-39da-43f0-8370-83ca99f1dee2.png">



### 2. Jenkins 브랜치 별 빌드(매개변수 이용)

```
이 빌드는 매개변수가 있습니다. 체크 후 String Parameter 추가 >> 매개변수 명 작성 >> branch specifier ${매개변수 명} 작성
```

<img src = "https://user-images.githubusercontent.com/28687900/159517992-cb2db1cb-427b-4fcb-a1c2-c09e8e79739b.png">

<img src = "https://user-images.githubusercontent.com/28687900/159518186-69d05288-1615-4c39-9971-b0405d9fb4f3.png">


```
Build with Parameter 기능 추가 된거 확인 >> 원하는 branch name 작성 후 빌드
```
<img src = "https://user-images.githubusercontent.com/28687900/159518344-5cda5c04-9303-4bd0-a673-b6a2f2d6251c.png">


___

