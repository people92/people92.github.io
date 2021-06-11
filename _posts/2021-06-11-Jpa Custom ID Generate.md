# JPA Custom ID Generate + PostgreSQL

## 목차
1. PostgreSQL 시퀀스 키 생성 및 조회 방법
2. JPA 키 자동생성 방법
3. JPA Custom ID Generate 예제

___

## __개요__
PostgreSQL과 JPA 적용 중 ID를 커스텀마이징 해서 만들어야 하는 일이 생겼다.  
이전에 써본 경험이 있는데 그 경험을 토대로 새롭게 적용해보려고 한다.
___

## __상세__

### 1. PostgreSQL 시퀀스 키 조회 방법
일단, PostgreSQL 디비는 시퀀스키 생성은 오라클과 같지만,  
조회하거나 증가시키는 쿼리는 다르다.

`Oracle`
``` oracle
select seq_xx.currval from dual; -- 시퀀스 현재값
select seq_xx.nextval from dual; -- 시퀀스 다음값 : 증가시키고 다음값 리턴
```

`PostgreSQL`
``` postgreSQL
select currval(seq_xx); -- 시퀀스 현재값
select nextval(seq_xx); -- 시퀀스 다음값 : 증가시키고 다음값 리턴
```

spring-boot에서 postgresql db를 사용하기 위해 dependency 추가
``` xml
       <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
```

### 2. JPA 키 자동생성 방법

JPA에서 기본 키 값이 자동으로 생성하려면 @GeneratedValue 어노테이션을 추가해야한다.  
기본키 자동생성 전략에는 4가지가 있다.(AUTO, IDENTITY, SEQUENCE, TABLE)
+ AUTO : 디폴트 값, '@GeneratedValue'는 '@GeneratedValue(strategy = GenerationType.AUTO)'와 같다.  JPA 구현체가 자동으로 생성전략 결정
+ IDENTITY : 기본키 생성을 데이터베이스에 위임
+ SEQUENCE : 데이터베이스의 특정 시퀀스를 사용하여 생성
+ TABLE : 데이터베이스의 키 생성 테이블을 만들고, 이를 사용하여 생성  

&#8251; Hibernate 5에 UUIDGenerator기능이 추가되었다. Hibernate는“8dd5f315-9788-4d00-87bb-10eed9eff566”형식의 ID를 생성
``` java
@Entity
public class Course {

    @Id
    @GeneratedValue
    private UUID courseId;

    // ...
}
```

### 3. Jpa Custom ID Generate 예제
키 자동생성 전략이 있지만, 생성방법을 커스텀하기위해 IdentifierGenerator 인터페이스를 구현하여 사용자지정생성기를 구현하였다.


``` java
package com.ssg.people92.jpa_postgresql.store.jpo;

import org.hibernate.HibernateException;
import org.hibernate.engine.spi.SharedSessionContractImplementor;
import org.hibernate.id.IdentifierGenerator;

import java.io.Serializable;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;

/*
* 사용자정의 키 생성
* */
public class CustomIdGenerator implements IdentifierGenerator {
    @Override
    public Serializable generate(SharedSessionContractImplementor sharedSessionContractImplementor, Object o) throws HibernateException {

        Connection connection = sharedSessionContractImplementor.connection();
        try {
            Statement statement = connection.createStatement();

            ResultSet rs = statement.executeQuery("select nextval('seq_payment')");

            if(rs.next())
            {
                int id = rs.getInt(1);
                StringBuffer sb = new StringBuffer();

                String generatedId = sb.append("TE").append(id).toString();

                return generatedId;
            }
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return null;
    }
}

```

처음에는 위에 소스처럼 키 접두사에 특정 문자열들을 추가하여 여러개의 Generator들을 만들었지만,
공부하다보니 문자열 접두사를 받아와 처리할 수 있는 방법이 있어서 그 방법으로 다시 작성해보았다.

1. CustomIdGenerator 구현시, IdentifierGenerator와 Configurable 인터페이스를 구현한다.
2. Configurable 인터페이스의 configure 메소드를 구현하여 jpa 매핑객체에서 넘겨준 prefix 파라미터 값을 가져온다.  
3. IdentifierGenerator 인터페이스의 generate 메소드에서 prefix별로 키 생성 방법을 별도로 작성하였다.


`Custom ID Generator`
``` java
package com.ssg.people92.jpa_postgresql.store.jpo;


import com.ssg.people92.jpa_postgresql.domain.entity.SequencePrefixConstants;
import org.hibernate.HibernateException;
import org.hibernate.MappingException;
import org.hibernate.engine.spi.SharedSessionContractImplementor;
import org.hibernate.id.Configurable;
import org.hibernate.id.IdentifierGenerator;
import org.hibernate.service.ServiceRegistry;
import org.hibernate.type.Type;

import java.io.Serializable;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.Properties;


public class MultiCustomIdGenerator implements IdentifierGenerator, Configurable {

    private String prefix;

    @Override
    public void configure(Type type, Properties properties, ServiceRegistry serviceRegistry) throws MappingException {
        this.prefix = properties.getProperty("prefix");
    }

    @Override
    public Serializable generate(SharedSessionContractImplementor sharedSessionContractImplementor, Object o) throws HibernateException {

        Connection connection = sharedSessionContractImplementor.connection();
        try {
            Statement statement = connection.createStatement();

            String query = "";

            if(SequencePrefixConstants.PAYMENT_KEY.equals(prefix)){
                query = "select nextval('seq_payment')";
            }else if(SequencePrefixConstants.SELLER_KEY.equals(prefix)){
                query = "select nextval('seq_seller')";
            }

            ResultSet rs = statement.executeQuery(query);

            if(rs.next())
            {
                int id = rs.getInt(1);
                StringBuffer sb = new StringBuffer();

                String generatedId = sb.append(prefix).append(id).toString();

                return generatedId;
            }
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return null;
    }
}
```

` JPA 객체` : @GenericGenerator 어노테이션을 이용해 parameter에 prefix에 값을 세팅해주고, strategy에 구현한 키 Generator 객체를 매핑해주면 된다.
``` java

    @Id
    @Column(name = "SELLER_ID")
    @GeneratedValue(generator = "sellerId-generator")
    @GenericGenerator(name = "sellerId-generator",
            parameters = @org.hibernate.annotations.Parameter(name = "prefix", value = SequencePrefixConstants.SELLER_KEY),
            strategy = "com.ssg.people92.jpa_postgresql.store.jpo.MultiCustomIdGenerator")
    private String sellerId;

```

` Junit 테스트 코드 `
``` java
package com.ssg.people92.jpa_postgresql;

import com.ssg.people92.jpa_postgresql.store.jpo.PaymentJpo;
import com.ssg.people92.jpa_postgresql.store.jpo.SellerJpo;
import com.ssg.people92.jpa_postgresql.store.repository.PaymentRepository;
import com.ssg.people92.jpa_postgresql.store.repository.SellerRepository;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class JpaCustomKeyTest {

    @Autowired
    PaymentRepository paymentRepository;

    @Autowired
    SellerRepository sellerRepository;

    @Test
    void savePaymentCustomKeyTest() {
        PaymentJpo paymentJpo = new PaymentJpo();
        paymentJpo.setMbrId("TEST1");
        paymentJpo.setPmtCode("TEST1");
        PaymentJpo result = paymentRepository.save(paymentJpo);

        Assertions.assertNotNull(result.getPmtId());
        Assertions.assertEquals(result.getPmtId().substring(0,2), "PM");
    }
    @Test
    void saveSellerCustomKeyTest() {
        SellerJpo sellerJpo = new SellerJpo();
        sellerJpo.setName("TEST1");
        SellerJpo result = sellerRepository.save(sellerJpo);

        Assertions.assertNotNull(result.getSellerId());
        Assertions.assertEquals(result.getSellerId().substring(0,2), "SE");
    }
}
```
참고 github 소스 : https://github.com/people92/jpa-postgresql  


___


## __참고__
https://www.baeldung.com/hibernate-identifiers  
https://velog.io/@modsiw/JPA-Hibernate%EA%B0%80-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%90%EB%8F%99-%ED%82%A4-%EC%83%9D%EC%84%B1-%EC%A0%84%EB%A0%A5%EC%9D%84-%EA%B2%B0%EC%A0%95%ED%95%98%EB%8A%94%EA%B0%80

