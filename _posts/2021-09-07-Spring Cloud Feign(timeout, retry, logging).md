# Spring Cloud Feign 추가 (timeout, retry, logging)

1. Feign client Timeout 설정 방법  

[global]

``` xml
feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic
```

※ feignClient 별로 처리도 가능  
``` xml
feign:
  client:
    config:
      feignName: #FeignClient에서 name 설정값으로 준 값
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic
```


2. Feign logging : loggerLevel로 위에 예시처럼 Clinet별로 처리 가능
- Feign logging은 DEBUG에서만 동작   
```
logging.level.com.study.springcloud.KakaoOpenApiClient: DEBUG # Client Class packages 경로
```
- 각 logging Type별 종류 및 결과 예시
1. NONE : 로그 없음  

2. BASIC : 요청 Method, URL, 응답상태코드, 실행시간만 출력  
```
2021-09-07 17:12:28.909 DEBUG 13064 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> GET https://dapi.kakao.com/v2/search/web?query=SSG HTTP/1.1
2021-09-07 17:12:29.342 DEBUG 13064 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] <--- HTTP/1.1 200 OK (430ms)
```

3. HEADERS : 요청/응답 헤더 출력  
```
2021-09-07 17:13:11.945 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> GET https://dapi.kakao.com/v2/search/web?query=SSG HTTP/1.1
2021-09-07 17:13:11.945 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] Authorization: KakaoAK 25ca90303af8723009e137e52babb4be
2021-09-07 17:13:11.945 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> END HTTP (0-byte body)
2021-09-07 17:13:12.160 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] <--- HTTP/1.1 200 OK (212ms)
2021-09-07 17:13:12.160 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] access-control-allow-headers: Authorization, KA, Origin, X-Requested-With, Content-Type, Accept
2021-09-07 17:13:12.160 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] access-control-allow-methods: GET, OPTIONS
2021-09-07 17:13:12.160 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] access-control-allow-origin: *
2021-09-07 17:13:12.160 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] connection: keep-alive
2021-09-07 17:13:12.160 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] content-type: application/json;charset=UTF-8
2021-09-07 17:13:12.160 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] date: Tue, 07 Sep 2021 08:13:11 GMT
2021-09-07 17:13:12.160 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] server: nginx
2021-09-07 17:13:12.160 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] transfer-encoding: chunked
2021-09-07 17:13:12.160 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] vary: Accept-Encoding
2021-09-07 17:13:12.161 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] x-request-id: 7177ec10-0fb3-11ec-81dc-59a271f11b60
2021-09-07 17:13:12.161 DEBUG 16316 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] <--- END HTTP (5709-byte body)
```

4. FULL : 요청/응답 헤더, 바디, 메타데이터 모두 출력  
```
2021-09-07 17:14:18.654 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> GET https://dapi.kakao.com/v2/search/web?query=SSG HTTP/1.1
2021-09-07 17:14:18.654 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] Authorization: KakaoAK 25ca90303af8723009e137e52babb4be
2021-09-07 17:14:18.654 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> END HTTP (0-byte body)
2021-09-07 17:14:18.866 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] <--- HTTP/1.1 200 OK (209ms)
2021-09-07 17:14:18.866 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] access-control-allow-headers: Authorization, KA, Origin, X-Requested-With, Content-Type, Accept
2021-09-07 17:14:18.866 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] access-control-allow-methods: GET, OPTIONS
2021-09-07 17:14:18.866 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] access-control-allow-origin: *
2021-09-07 17:14:18.866 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] connection: keep-alive
2021-09-07 17:14:18.866 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] content-type: application/json;charset=UTF-8
2021-09-07 17:14:18.866 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] date: Tue, 07 Sep 2021 08:14:18 GMT
2021-09-07 17:14:18.866 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] server: nginx
2021-09-07 17:14:18.866 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] transfer-encoding: chunked
2021-09-07 17:14:18.866 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] vary: Accept-Encoding
2021-09-07 17:14:18.866 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] x-request-id: 993a2510-0fb3-11ec-988a-9b1b68270388
2021-09-07 17:14:18.867 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] 
2021-09-07 17:14:18.867 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] {"documents":[{"contents":"네이버페이 · 카카오페이 · 페이코 · PayPal 물류유통 서비스사 스마일페이 · SK페이 · L.pay · \u003cb\u003eSSG\u003c/b\u003e PAY 핀테크 서비스사 Paynow · 차이 · 토스 · 머니트리 교통카드사 티머니페이 증권사 미래에셋페이...","datetime":"2021-09-04T00:00:00.000+09:00","title":"\u003cb\u003eSSG\u003c/b\u003e PAY - 나무위키","url":"https://namu.wiki/w/SSG%20PAY"},{"contents":"2021년 시즌에 대한 내용은 \u003cb\u003eSSG\u003c/b\u003e 랜더스/2021년 문서 , 2021시즌 현황에 대한 내용은 \u003cb\u003eSSG\u003c/b\u003e 랜더스/2021년/9월 문서 참고하십시오. KBO 리그 소속 구단 [ 펼치기 · 접기 ] NC 다이노스 두산 베어스 kt wiz LG...","datetime":"2021-09-02T00:00:00.000+09:00","title":"\u003cb\u003eSSG\u003c/b\u003e 랜더스 - 나무위키","url":"https://namu.wiki/w/SSG%20%EB%9E%9C%EB%8D%94%EC%8A%A4"},{"contents":"\u003cb\u003eSSG\u003c/b\u003e 랜더스가 4연승에 도전한다. \u003cb\u003eSSG\u003c/b\u003e는 4일 고척스카이돔 원정을 떠나 키움 히어로즈와 시즌 13차전을 펼친다. 전날(3일) 두산 베어스와 인천 홈경기에서 3-1 승리를 거두며 3연승에 성공한 \u003cb\u003eSSG\u003c/b\u003e가 연승 행진을...","datetime":"2021-09-07T12:51:46.000+09:00","title":"[4일 프리뷰] 방망이 탄력 받은 \u003cb\u003eSSG\u003c/b\u003e, 오원석 올려 4연승 도전","url":"https://www.koreabaseball.com/News/Preview/View.aspx?bdSe=48656"},{"contents":"KT 위즈가 투타 조화를 앞세워 \u003cb\u003eSSG\u003c/b\u003e 랜더스를 4연패에 빠트렸다. KT 위즈는 25일 수원KT위즈파크에서 열린 2021 신한은행 SOL KBO리그 \u003cb\u003eSSG\u003c/b\u003e 랜더스와의 시즌 8차전에서 7-1로 승리했다. 선두 KT는 3연전 기선을...","datetime":"2021-09-03T06:48:17.000+09:00","title":"\u0026#39;소형준 완벽투\u0026amp;황재균 3안타\u0026#39; KT, \u003cb\u003eSSG\u003c/b\u003e 7-1 완파…\u003cb\u003eSSG\u003c/b\u003e 4연패","url":"https://www.koreabaseball.com/News/BreakingNews/View.aspx?bdSe=47507"},{"contents":"프로야구 KBO리그 \u003cb\u003eSSG\u003c/b\u003e 랜더스와 KT 위즈의 경기가 25일 오후 수원 kt위즈파크에서 열렸다. KT 선발 소형준이 힘차게 공을 던지고 있다. 수원=정시종 기자 ****.*******@********.**.** /2021.08.25. 소형준(20...","datetime":"2021-08-25T20:30:59.000+09:00","title":"KT 소형준, \u003cb\u003eSSG\u003c/b\u003e전 6이닝 1실점...시즌 4승 요건","url":"http://isplus.live.joins.com/news/article/article.asp?total_id=24134772\u0026ctg=1401\u0026tm=i_b\u0026tag="},{"contents":"확보한 자금 물류 인프라 및 IT 분야 집중 투자 \u003cb\u003eSSG\u003c/b\u003e닷컴은 상장 주관사를 선정하고자 주요 증권사에 입찰제안요청서(RFP)를 발송했다고 13일 밝혔다. (사진=\u003cb\u003eSSG\u003c/b\u003e닷컴 홈페이지 캡처) [화이트페이퍼=이시아 기자...","datetime":"2021-08-13T12:26:41.000+09:00","title":"\u003cb\u003eSSG\u003c/b\u003e닷컴 상장 절차 본격 시동… 주관사 선정 돌입","url":"https://www.whitepaper.co.kr/news/articleView.html?idxno=208765"},{"contents":"for 로저드뷔 최근 KBO 리그(대한민국 프로야구 리그)는 신세계 그룹(이마트)의 신생 야구단 \u003cb\u003eSSG\u003c/b\u003e 랜더스(\u003cb\u003eSSG\u003c/b\u003e Landers, 구 SK 와이번스) 소식으로 연일 뜨겁습니다. 특히 메이저리거 추신수 선수(외야수, 17번)의...","datetime":"2021-09-01T17:28:49.000+09:00","title":"\u003cb\u003eSSG\u003c/b\u003e 랜더스에 합류한 추신수 선수의 로저드뷔 시계 - TIMEFORUM - NEWS - TIMEFORUM","url":"https://www.timeforum.co.kr/NEWSNINFORMATION/19083806"},{"contents":"사랑=최한슬 기자] 컴퓨터 게이밍의자 키보드 마우스 책상 이어폰 브랜드 제닉스가 지난 30일부터 \u003cb\u003eSSG\u003c/b\u003e닷컴에서 아레나 엑스 제로(ARENA-X ZERO) 레드 게이밍 의자를 총 150대만 최대 46% 할인된 가격으로 판매...","datetime":"2021-09-01T14:28:12.000+09:00","title":"제닉스, \u003cb\u003eSSG\u003c/b\u003e닷컴서 \u0026#39;아레나-X 제로 게이밍 의자\u0026#39; 최대 46% 할인 한정 판매 돌입한다","url":"http://www.ilovepc.co.kr/news/articleView.html?idxno=40544"},{"contents":"선제적으로 관련 인프라를 준비해온 조직들은 보다 유리한 환경에서 비즈니스를 영위해나가고 있다. \u003cb\u003eSSG\u003c/b\u003e닷컴(\u003cb\u003eSSG\u003c/b\u003e.COM)은 이러한 데이터의 중요성을 선제적으로 인식하고 대응해온 기업 중 하나다. 상품 검색...","datetime":"2021-09-06T20:24:30.000+09:00","title":"[구축사례] \u003cb\u003eSSG\u003c/b\u003e닷컴, 데이터 관리도 ‘쓱(\u003cb\u003eSSG\u003c/b\u003e)’","url":"http://www.comworld.co.kr/news/articleView.html?idxno=50178"},{"contents":"프로야구 \u003cb\u003eSSG\u003c/b\u003e 조웅천 1군 메인 투수 코치가 신종 코로나바이러스 감염증(코로나19) 확진자와 밀접접촉해 2주간 자가격리한다. 2021프로야구 KBO리그 \u003cb\u003eSSG\u003c/b\u003e랜더스와 LG트윈스의 경기가 22일 오후 인천 \u003cb\u003eSSG\u003c/b\u003e랜더스필드...","datetime":"2021-08-11T17:41:59.000+09:00","title":"\u003cb\u003eSSG\u003c/b\u003e 조웅천 투수 코치, 밀접접촉자로 자가격리","url":"http://isplus.live.joins.com/news/article/article.asp?total_id=24125947\u0026ctg=1401\u0026tm=i_b\u0026tag="}],"meta":{"is_end":false,"pageable_count":435,"total_count":516489}}
2021-09-07 17:14:18.867 DEBUG 13812 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] <--- END HTTP (5709-byte body)
```

3. Feign retry  
- Spring Cloud Feign retry 기능을 포함하고 있다. 하지만 기본적으로는 default로 retry off가 되어있다.  
- 따라서, retry 기능을 사용하려면 아래와 같이 Feign Retry 빈을 등록해주어야 함.
``` java
@Configuration
public class FeignRetryConfiguration {
    @Bean
    public Retryer retryer() {
        return new Retryer.Default(100, 2000, 3);
    }
}

```
- Retryer.Default() parameter 설명  
  + 1번째 인자 - period : 각 시도간의 차이
  + 2번째 인자 - maxPeriod : 모든 재시도 사이의 시간
  + 3번째 인자 - maxAttempts : 최대 시도수

결과 : retry 3회 처리 후에 fallback 처리되는것을 확인할수 있음.  

```
2021-09-07 17:16:07.574 DEBUG 8412 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> GET https://dapi.kakao.com/v2/search/web?query=SSG HTTP/1.1
2021-09-07 17:16:07.656 DEBUG 8412 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] <--- ERROR SocketTimeoutException: connect timed out (82ms)
2021-09-07 17:16:07.821 DEBUG 8412 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> RETRYING
2021-09-07 17:16:07.821 DEBUG 8412 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> GET https://dapi.kakao.com/v2/search/web?query=SSG HTTP/1.1
2021-09-07 17:16:07.837 DEBUG 8412 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] <--- ERROR SocketTimeoutException: connect timed out (15ms)
2021-09-07 17:16:08.062 DEBUG 8412 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> RETRYING
2021-09-07 17:16:08.062 DEBUG 8412 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] ---> GET https://dapi.kakao.com/v2/search/web?query=SSG HTTP/1.1
2021-09-07 17:16:08.073 DEBUG 8412 --- [akao-open-api-1] c.study.springcloud.KakaoOpenApiClient   : [KakaoOpenApiClient#searchDaumWeb] <--- ERROR SocketTimeoutException: connect timed out (10ms)
2021-09-07 17:16:08.078 ERROR 8412 --- [akao-open-api-1] c.s.s.KakaoOpenApiClientFallbackFactory  : Feign Client Error : connect timed out executing GET https://dapi.kakao.com/v2/search/web?query=SSG
```

