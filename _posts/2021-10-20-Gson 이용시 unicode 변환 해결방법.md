---
layout: post
title:  "Gson 이용 시 Unicode 변환 해결 방법"
date:   2021-10-20
excerpt: "Gson 이용 시 Unicode 변환 해결 방법"
tag:
- markdown 
comments: false
---


# Gson 이용 시 Unicode 변환 해결 방법

## 목차
1. Unicode란?
2. Gson 사용시 unicode 문제
3. Gson 사용시 unicode 해결방법

___

## __개요__
데이터 처리 중 암호화된 데이터 값이 달라지는 걸 발견하였는데,  
알고보니 객체를 json으로 변환 중 문자가 unicode로 변환되어 나오는 것을 확인하였고,
해결방법에 대해서 알아보려고 한다.

___

## __상세__

### 1. unicode란?
`Unicode 정의`
> 유니코드(영어: Unicode)는 전 세계의 모든 문자를 컴퓨터에서 일관되게 표현하고 다룰 수 있도록 설계된 산업 표준이다. 유니코드는 유니코드 협회(Unicode Consortium)가 제정한다. 또한 이 표준에는 ISO 10646 문자 집합, 문자 인코딩, 문자 정보 데이터베이스, 문자들을 다루기 위한 알고리즘 등을 포함하고 있다.



### 2. Gson 사용시 unicode 문제


``` java
    @Test
    public void gsonUnicodeTest() {
        Map<String, String> map = new HashMap<>();
        map.put("unicode", "7456156==");
        System.out.println(JsonUtils.toJson(map));
    }
```

``` 
{"unicode":"7456156\u003d\u003d"}
```

-> Gson을 이용하여 Json 변환 시, 문자 '='가 '\u003d' 유니코드로 변환된다. 


### 3. Gson 사용시 unicode 해결방법

`` 기존 : 일반적인 Gson 객체 생성``
``` java
    public static <T> String toJson(T obj) {
        Gson gson = new Gson();
        return gson.toJson(obj);
    }

```

`` 변경 : GsonBuilder().disableHtmlEscaping().create() 사용하여 Gson 객체 생성``
``` java
    public static <T> String toJsonWithoutUnicode(T obj) {
        Gson gson = new GsonBuilder().disableHtmlEscaping().create();
        return gson.toJson(obj);
    }
```
- GsonBuilder().disableHtmlEscaping().create() 사용하여 HTML Escape 비활성화하면 unicode로 변환하지 않고 정상처리 된다.

```
{"unicode":"7456156=="}
```

___


## __참고__
https://www.javadoc.io/doc/com.google.code.gson/gson/2.8.5/com/google/gson/GsonBuilder.html#disableHtmlEscaping--  
