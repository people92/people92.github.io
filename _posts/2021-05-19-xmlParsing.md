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
추가 예정

___


## __참고__
http://tcpschool.com/json/json_intro_xml  
https://nowonbun.tistory.com/561  
https://jeong-pro.tistory.com/144  
https://12bme.tistory.com/202