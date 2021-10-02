---
layout: post
title:  "[Java] Json File Download(HttpURLConnection, FeignClient)"
date:   2021-10-02
excerpt: "[Java] Json File Download(HttpURLConnection, FeignClient)"
tag:
- markdown 
comments: false
---

# [Java] Json File Download(HttpURLConnection, FeignClient)

## 목차
1. HttpURLConnection 이용한 파일 다운로드
2. FeignClient 이용한 파일 다운로드


___


## __개요__
다른 업체에서 json 파일을 서버에 저장해놓고 다운받아서 처리하는 경우가 많은데  
이 경우 api로 호출해서 내 로컬(서버)에 저장하는 방식에 대해서 알아보려고 한다.

___

## __상세__

### 1. HttpURLConnection 이용한 파일 다운로드

- InputStream : 파일 데이터를 읽거나 네트워크 소켓을 통해 데이터를 읽거나 키보드에서 입력한 데이터를 읽을 때 사용  
    + read(byte[] b): 배열 b의 크기만큼 데이터를 읽어와서 b에 저장한다.
    + read(byte[] b, int off, int len) : len의 크기만큼 데이터를 읽어와서 배열 b의 off 위치부터 저장한다.   


- FileOutputStream : 데이터를 파일에 바이트 스트림으로 저장하기 위해 사용한다.   
                     주어진 이름의 파일을 쓰기 위한 객체를 생성하여 파일 생성됨  
    + write(byte[] b) : 배열 b에 저장된 모든 내용을 출력소스에 쓴다
    + write(byte[] b, int off, int len) : 배열 b에 저장된 내용을 off 위치부터 len개 만큼 출력소스에 쓴다.


&#8251; HttpURLConnection의 getInputStream() 메소드를 통해 응답 데이터를 읽을 수 있는 InputStream 객체를 얻을 수 있습니다.   

&#8251; InputStream read시 더 이상 읽은 바이트가 없으면 -1 리턴  

``` java
  public void downloadFile(String requestUrl) {
        String outputDir = "C:/Code/file";
        InputStream is = null;
        FileOutputStream os = null;
        try{
            URL url = new URL(requestUrl);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            int responseCode = conn.getResponseCode();

            if (responseCode == HttpURLConnection.HTTP_OK) {
                String fileName = "downloadTest" + System.currentTimeMillis() + ".json";
                System.out.println("fileName = " + fileName);
                File file = new File(outputDir);

                if(!file.exists()) file.mkdirs();

                is = conn.getInputStream();
                os = new FileOutputStream(file + "/" + fileName);

                int bytesRead;
                byte[] buffer = new byte[1024];
                while ((bytesRead = is.read(buffer)) != -1) {
                    // 입력받은 내용을 파일 내용으로 기록한다.
                    os.write(buffer, 0, bytesRead);
                }

            } else {
                System.out.println("fail => responseCode : " + responseCode);
            }
            conn.disconnect();
        } catch (Exception e){
            e.printStackTrace();
        } finally {
            try {
                if (is != null){
                    is.close();
                }
                if (os != null){
                    os.close();
                }
            } catch (IOException e){
                e.printStackTrace();
            }
        }
    }
```


### 2. FeignClient 이용한 파일 다운로드  
1. FeignClient 호출시 리턴 타입을 feign에서 제공해주는 Reponse 객체로 받는다.  
2. Feign.Reponse.Body 객체 안에 있는 asInputStream 메소드를 통해 InputStream 객체를 얻을수 있음.
3. 그 이후 파일 다운로드 방법은 위의 HttpURLConnection 사용 시 예제와 유사

``` java
@FeignClient(name = "download-api",
        url = "https://raw.githubusercontent.com/people92/self-study/main/java-study/src/main/resources/json"
)
public interface DownloadFeignClient {

    @GetMapping(value = "/sample.json")
    Response downloadJsonFile();
}
```


``` java
    @Test
    public void downloadJsonFileTest() throws IOException {

        String outputDir = "C:/Code/file";
        String fileName = "downloadTest" + System.currentTimeMillis() + ".json";

        Response response = downloadFeignClient.downloadJsonFile();
        Response.Body body = response.body();
        InputStream inputStream = body.asInputStream();

        if(response.status() == 200) {
            FileOutputStream os = new FileOutputStream(outputDir + "/" + fileName);
            int bytesRead;
            byte[] buffer = new byte[1024];
            while ((bytesRead = inputStream.read(buffer)) != -1) {
                // 입력받은 내용을 파일 내용으로 기록한다.
                os.write(buffer, 0, bytesRead);
            }
            inputStream.close();
            os.close();
        }
    }
```
___


## __참고__
https://velog.io/@hellonewtry/Java-%EC%9E%90%EB%B0%94-HttpUrlConnection-%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-%ED%8C%8C%EC%9D%BC-%EB%8B%A4%EC%9A%B4%EB%B0%9B%EA%B8%B0   
https://stackoverflow.com/questions/59765206/feign-for-downloading-file