# Spring Bean Injection + Factory Method Design Pattern

## 목차
1. 팩토리 메소드 디자인 패턴 정의
2. 팩토리 메소드 디자인 패턴을 이용해 동적으로 빈 주입 예제
___

## __개요__
개발 진행 중 하나의 인터페이스를 가지고 여러개의 구현체를 구현해야 되는 상황이 생겼다.  
이전의 팩토리 메소드 패턴을 이용하여 빈 주입하는 소스를 인수인계 받은 적이 있는데,  
그 방법에 대해서 예제를 작성해보려고 한다.

___

## __상세__

### 1. 팩토리 메소드 디자인 패턴 정의

#### 팩토리 메소드 디자인 패턴(Factory Method Design Pattern)
> 객체를 생성하기 위한 인터페이스를 정의하고, 어떤 클래스의 인스턴스를 생성할지에 대한 처리는 서브클래스가 결정하는 디자인 패턴 - GoF

#### 팩토리 메소드 패턴 사용 이유
- 추가될 클래스가 있을 시, 기존 코드 수정이 필요없이 신규 클래스만 추가되면 되므로 결합도가 낮다.
  

### 2. 팩토리 메소드 디자인 패턴을 이용해 동적으로 빈 주입 예제
하나의 인터페이스에 여러개의 구현체가 있을 경우,  
매번 구현클래스를 다르게 선언하지 않고, 팩토리 메소드 디자인 패턴을 이용하여 빈 객체를 동적으로 가져와 사용할 수 있다.

> Spring에서는 Bean을 Collection으로 주입할(Injection) 수 있다.   
> 다시 말해, 하나의 인터페이스를 상속받고 있는 구현체들인 bean들을 List로 주입할 수 있다.

예제 코드를 작성해보겠다.

우선, 주식 회사의 한국 이름을 가져오는 StockService Interface가 있다.  
getStockGroup 메소드는 구현된 클래스를 구별하기 위한 return이 enum 클래시인 메소드이다.
``` java
public interface StockService {

    StockGroup getStockGroup();

    String findKoreanName();
}
```

주식 회사의 종류는 애플, 네이버, 에어비앤비, 삼성 4가지로 Enum 타입을 이용해 작성하였다.
``` java
public enum StockGroup {

    APPLE("애플","1001"),
    NAVER("네이버","1002"),
    AIRBNB("에어비앤비","1003"),
    SAMSUNG("삼성","1004"),
    EMPTY("없음", "0000");

    private String koreanName;
    private String stockId;

    StockGroup(String koreanName, String stockId){
        this.koreanName = koreanName;
        this.stockId = stockId;
    }

    public static StockGroup findByStockId(String stockId) {
        return Arrays.stream(StockGroup.values())
                .filter(stockGroup -> stockId.equals(stockGroup.stockId))
                .findAny().orElse(EMPTY);
    }

    public String getKoreanName() {
        return koreanName;
    }
}
```

StockService를 구현하는 클래스는 회사별로 총 4가지 클래스로 구현되어있다.  
각 클래스마다 StockGroup Enum 클래스를 리턴해주는 메소드, 한글 이름을 리턴해주는 메소드로 작성하였다.
``` java
@Service
public class AirbnbServiceImpl implements StockService{
    @Override
    public StockGroup getStockGroup() {
        return StockGroup.AIRBNB;
    }

    @Override
    public String findKoreanName() {
        return StockGroup.AIRBNB.getKoreanName();
    }
}
@Service
public class AppleServiceImpl implements StockService{
    @Override
    public StockGroup getStockGroup() {
        return StockGroup.APPLE;
    }

    @Override
    public String findKoreanName() {
        return StockGroup.APPLE.getKoreanName();
    }
}
@Service
public class NaverServiceImpl implements StockService{
    @Override
    public StockGroup getStockGroup() {
        return StockGroup.NAVER;
    }

    @Override
    public String findKoreanName() {
        return StockGroup.NAVER.getKoreanName();
    }
}
@Service
public class SamsungServiceImpl implements StockService{
    @Override
    public StockGroup getStockGroup() {
        return StockGroup.SAMSUNG;
    }

    @Override
    public String findKoreanName() {
        return StockGroup.SAMSUNG.getKoreanName();
    }
}

```

 &#8251; 이렇게 회사마다 클래스를 다르게 구현하였는데, 실제 다른 회사가 추가 된다면 추가된 다른 회사의 클래스만 구현하도록 하면 된다.

실제 구현하는 클래스들의 빈들을 Map에 담아 주입한다.  
이때 Map의 Key는 위에서 선언한 StockGroup의 Enum 클래스이고, Value는 StockService 인터페이스이다.  
Map에 인터페이스를 구현하는 클래스들을 모두 담고, getService라는 메소드를 이용해 키값 별로 원하는 클래스들에 접근할 수 있다.

``` java
@Component
public class StockServiceFactory {

    private final Map<StockGroup, StockService> serviceMap = new HashMap<>();

    public StockServiceFactory(List<StockService> stockServices) {

        for(StockService stockService : stockServices) {
            this.serviceMap.put(stockService.getStockGroup(), stockService);
        }
    }

    public StockService getSerivce(StockGroup stockGroup) {
        return this.serviceMap.get(stockGroup);
    }
}
```


팩토리 메서드 패턴을 이용하기 위해 생성한 -Factory.class를 주입받아서 사용한다.  

`Test 코드`
``` java
@SpringBootTest
public class FactoryPatternTest {

    @Autowired
    StockServiceFactory stockServiceFactory;

    @Test
    public void beanInjectionTest() {
        StockService appleService = stockServiceFactory.getSerivce(APPLE);
        StockService airbnbService = stockServiceFactory.getSerivce(AIRBNB);
        StockService samsungService = stockServiceFactory.getSerivce(SAMSUNG);
        StockService naverService = stockServiceFactory.getSerivce(NAVER);

        Assertions.assertEquals(appleService.getStockGroup(), APPLE);
        Assertions.assertEquals(airbnbService.getStockGroup(), AIRBNB);
        Assertions.assertEquals(samsungService.getStockGroup(), SAMSUNG);
        Assertions.assertEquals(naverService.getStockGroup(), NAVER);
    }
    @Test
    public void getKoreanNameTest() {
        String appleKoreanName = stockServiceFactory.getSerivce(APPLE).findKoreanName();
        String airbnbKoreanName = stockServiceFactory.getSerivce(AIRBNB).findKoreanName();
        String samsungKoreanName = stockServiceFactory.getSerivce(SAMSUNG).findKoreanName();
        String naverKoreanName = stockServiceFactory.getSerivce(NAVER).findKoreanName();

        Assertions.assertEquals(appleKoreanName, APPLE.getKoreanName());
        Assertions.assertEquals(airbnbKoreanName, AIRBNB.getKoreanName());
        Assertions.assertEquals(samsungKoreanName, SAMSUNG.getKoreanName());
        Assertions.assertEquals(naverKoreanName, NAVER.getKoreanName());
    }
}
```

참고 소스 : https://github.com/people92/design-pattern  

___


## __참고__
https://zorba91.tistory.com/306  

