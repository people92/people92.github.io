---
layout: post
title:  "XML Parsing"
date:   2021-05-19
excerpt: "XML Parsing"
tag:
- markdown 
comments: false
---

# XML Parsing

## 목차


1. JSON vs XML
2. XML 사용 이유
3. XML Parsing 예제
4. XML 생성 예제


___

## __개요__
새로 개발해야 하는 업무에서 s기업하고 데이터를 주고 받아야 하는 업무를 개발해야 할 것 같다.  
기획 된 문서를 보니 서로 통신을 json이 아닌 xml로 주고 받기로 정해져있었다.  
요즘 json말고 xml쓰는 곳도 있구나 생각햿지만, 현업에서는 아직도 실제로 많이 사용되고 있었다.  
json으로 통신은 해보았지만 xml로는 해본적이 없어서 xml에 대해서 공부를 하고 파싱하는 법에 대해서 알아보려고 한다.

___

## __상세__

### 1. JSON vs XML
__XML이란?__  
XML은 EXtensible Markup Language의 약자입니다.  
XML은 HTML과 매우 비슷한 문자 기반의 마크업 언어(text-based markup language)입니다.  
이 언어는 사람과 기계가 동시에 읽기 편한 구조로 되어 있습니다.  
XML은 HTML처럼 데이터를 보여주는 목적이 아닌, 데이터를 저장하고 전달할 목적으로만 만들어졌습니다.  
또한, XML 태그는 HTML 태그처럼 미리 정의되어 있지 않고, 사용자가 직접 정의할 수 있습니다.  

__JSON이란?__  
JSON은 JavaScript Object Notation의 약자입니다.  
브라우저 통신을 위한 속성-값 또는 키-값 쌍으로 이루어진 데이터 포맷  

`json 예시`
``` json
{
    "name": "식빵",
    "family": "웰시코기",
    "age": 1,
    "weight": 2.14
}
```

### 2. XML 사용 이유
1. 일단 아직 실제 현업에서 자주 사용 
2. 신규 프로젝트들은 json을 이용하는 것이 좋지만, 굳이 기존 xml을 json으로 바꿀 필요는 없다.
3. 그래픽 파일이나 문서 등의 바이너리(Binary) 코딩된 파일은 xml이 더 적합

### 3. XML Parsing 예제
공공데이터포털에서 제공해주는 오픈 API를 이용해 XML parsing 예제를 개발해보려고 한다.  
버스노선 가져오는 오픈 API를 이용해 예제를 작성해보겠다.  

HttpUrlConnetion을 통해 오픈 API 호출해 xml 파싱해보는 예제를 작성해 보도록 하겠다.

xml 파싱하는 방법은 다양한 방법이 있지만, 그 중 XPath 라이브러리를 사용하여 파싱해보도록 하겠다.  
* XPath : 자바에서 내장 패키지로 제공하는 XML 파싱 라이브러리

[예제]
1. 공공데이터 오픈 API를 호출 해 xml 결과를 가져온다.
``` java
        String address = "http://openapitraffic.daejeon.go.kr/api/rest/busRouteInfo/getStaionByRoute?busRouteId=30300001&serviceKey=";
        String serviceKey = "ERvgM5YJuigXEYahj%2FZwKcD13pTrKUYUZOwWRuamCJfozqcMh4xyI9iMPTApsOQTzGcTeRJH7KIHmywWmYxe1g%3D%3D";

        URL url = new URL(address+serviceKey);
        HttpURLConnection con = (HttpURLConnection) url.openConnection();

        con.setRequestMethod("GET");

        int responseCode = con.getResponseCode();

        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(con.getInputStream()));

        String inputLine;
        StringBuffer response = new StringBuffer();

        while((inputLine = bufferedReader.readLine()) != null) {
            response.append(inputLine);
        }
        bufferedReader.close();

        return response.toString();
```

`open API xml 결과`  
<img src = "https://github.com/people92/people92.github.io/blob/master/img/openAPIResult.png?raw=true">


2. DocumentBuilderFactory 생성
``` java
DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory.newInstance();
```
3. DocumentBuilder 생성
``` java
DocumentBuilder documentBuilder = documentBuilderFactory.newDocumentBuilder();
```
4. InputSource 생성
``` java
InputSource is = new InputSource(new StringReader(xml));
```
5. XML 파싱
``` java
Document doc = documentBuilder.parse(is);
```
6. Xpath 객체 생성
``` java
XPath xPath = XPathFactory.newInstance().newXPath();
```
7. 데이터 추출 : 결과 xml에서 정류장 이름만 추출
``` java
        String expression = "//itemList/BUSSTOP_NM";

        NodeList nodeList = (NodeList) xPath.evaluate(expression, doc, XPathConstants.NODESET);

        for(int i = 0 ; i < nodeList.getLength(); i++) {
            System.out.println(nodeList.item(i).getTextContent());
        }
```

8. 결과  
<img src = "https://github.com/people92/people92.github.io/blob/master/img/parseResult.png?raw=true">


### 4. XML 생성 예제
JAXB(Java Architecture for XML Binding) API를 이용해 XML을 생성해보려고 한다.  
&#8251; JAXB : 자바 오브젝트를 XML로 변환하는 마샬링(Marshalling)을 가능하도록 하는 Java API  



- 어노테이션 참고   
@XmlRootElement(name = "XmlParent") : XML 특정 노드의 루트라는 의미, name 사용하여 명시 가능  
@XmlAccessorType(XmlAccessType.NONE) : @XmlElment 어노테이션으로 된 필드와 프로퍼티만 추출  
@XmlAccessorType(XmlAccessType.FIELD) : 모든 필드 추출  
@XmlAccessorType(XmlAccessType.PROPERTY) : 모든 프로퍼티 추출  
@XmlAccessorType(XmlAccessType.PUBLIC_MEMBER) : public 필드와 프로퍼티 추출  
@XmlElement(name = "XmlHeader") : XML 노드라는 의미, name 사용하여 명시 가능  


1. XML 생성을 위한 객체 생성 : XmlParent > XmlHeader, XmlBody


``` java
package com.server.people.xml.create;

import lombok.*;

import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement(name = "XmlParent")
@XmlAccessorType(XmlAccessType.FIELD)
@AllArgsConstructor
@Builder
@NoArgsConstructor
@Getter
@Setter
public class XmlParent {

    @XmlElement(name = "XmlHeader")
    private XmlHeader xmlHeader;

    @XmlElement(name = "XmlBody")
    private XmlBody xmlBody;


}
```


``` java
package com.server.people.xml.create;

import lombok.*;

import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement(name = "XmlHeader")
@XmlAccessorType(XmlAccessType.FIELD)
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Getter
@Setter
public class XmlHeader {

    @XmlElement(name = "sender")
    private String sender;

    @XmlElement(name = "receiver")
    private String receiver;

    @XmlElement(name = "mainId")
    private String mainId;
}
```


``` java
package com.server.people.xml.create;

import lombok.*;

import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement(name = "XmlBody")
@XmlAccessorType(XmlAccessType.FIELD)
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
public class XmlBody {

    @XmlElement(name = "code")
    private String code;

    @XmlElement(name = "message")
    private String message;

    @XmlElement(name = "sendDate")
    private String sendDate;
}

```

2. JAXB를 이용해 생성한 객체 마샬링

``` java
    public static String toXml(Object obj) throws Exception {

        JAXBContext jc = JAXBContext.newInstance(obj.getClass());

        Marshaller marshaller = jc.createMarshaller();

        marshaller.setProperty(Marshaller.JAXB_ENCODING, "utf-8");
        marshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, Boolean.TRUE);
        marshaller.setProperty(Marshaller.JAXB_FRAGMENT, true);

        StringWriter writer = new StringWriter();

        marshaller.marshal(obj, writer);

        return writer.toString();
    }
```

`테스트 코드`
``` java
package com.server.people.xml.parsing;

import com.server.people.xml.create.XmlBody;
import com.server.people.xml.create.XmlHeader;
import com.server.people.xml.create.XmlParent;
import com.server.people.xml.parsing.util.XmlUtils;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class XmlUtilsTest {

    @Test
    void xmlUtilsToXmlTest() throws Exception{

        XmlHeader xmlHeader = XmlHeader.builder().mainId("TEST1").receiver("re").sender("se").build();
        XmlBody xmlBody = XmlBody.builder().code("t1").message("TEST").build();

        XmlParent xmlParent = XmlParent.builder().xmlBody(xmlBody).xmlHeader(xmlHeader).build();

        System.out.println(XmlUtils.toXml(xmlParent));
    }
}

```

`결과 콘솔`  
원하는 트리구조로 xml 생성
```
<XmlParent>
    <XmlHeader>
        <sender>se</sender>
        <receiver>re</receiver>
        <mainId>TEST1</mainId>
    </XmlHeader>
    <XmlBody>
        <code>t1</code>
        <message>TEST</message>
    </XmlBody>
</XmlParent>
```


___



### 추가. xml to json 변환

maven dependency 추가
``` xml
<dependency>
  <groupId>org.json</groupId>
  <artifactId>json</artifactId>
  <version>20180813</version>
</dependency>
```

org.json 패키지에 있는 JSONObject와 XML 사용하여 변환
``` java
    public static String toJson(String xml) throws Exception {

        JSONObject jsonObject = XML.toJSONObject(xml);

        return jsonObject.toString();
        
    }
```

빌드하려고 하니 Found multiple occurrences of org.json.JSONObject 해당 에러가 발생하여 진행되지 않았다.  
로컬 레퍼지토리 .m2에 같은 이름의 클래스가 중복되어서 존재하여서 발생.  
-> 사용하지 않는 디팬던시 exclusion 제외하니깐 정상 작동

``` xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>com.vaadin.external.google</groupId>
                    <artifactId>android-json</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

`테스트 결과 콘솔` 
```
{"XmlParent":{"XmlHeader":{"receiver":"re","sender":"se","mainId":"TEST1"},"XmlBody":{"code":"t1","message":"TEST"}}}

```

참고소스 : https://github.com/people92/xml


___


## __참고__
http://tcpschool.com/json/json_intro_xml  
https://nowonbun.tistory.com/561  
https://jeong-pro.tistory.com/144  
https://12bme.tistory.com/202
