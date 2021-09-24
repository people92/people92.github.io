---
layout: post
title:  "JPA Auditing"
date:   2021-06-03
excerpt: "JPA Auditing"
tag:
- markdown 
comments: false
---


# JPA Auditing  

## 목차
1. JPA Auditing이란?
2. JPA Auditing 예제
3. JPA Auditing 테스트 결과


___

## __개요__

JPA를 이용하여 Audit 항목들을 다루어 보려고 한다.

___

## __상세__

### 1. JPA Auditing이란?
어플리케이션 개발을 하다보면 각 업무 환경에서 공통으로 사용하는 테이블 항목들이 존재한다.  
이때 변경사항을 추적하기 위한 필수적인 생성일자, 생성자, 수정일자, 수정자 등과 같은 공통적으로 사용되는 항목들이 존재하는데
이러한 항목들을 Audit 항목이라고 한다.  
Spring Data JPA에서는 Audit 항목들을 효율적으로 관리하고 매핑 할수 있도록 Auditing 기능을 제공하고 있다.

### 2. JPA Auditing 예제

1. @EnableJpaAuditing 어노테이션 추가하여 JPA Auditing 기능 활성화

``` java
@EnableJpaAuditing
public class T2021hwApplication {

	public static void main(String[] args) {
		SpringApplication.run(T2021hwApplication.class, args);
	}

}
```


2. Audit항목 관련 객체 클래스 생성
   - @MappedSuperclass : JPA에서 Table에 대한 공통 항목들이 존재할때 부모클래스에 정의하고 상속받아 중복제거
   - @EntityListeners : JPA DB 반영 이전, 이후에 사용자 정의 콜백 요청
   - AuditingEntityListener.class) : JPA 객체에 Auditing 정보를 제공
   - @CreatedBy : 데이터 생성자 자동생성
   - @CreatedDate : 데이터 생성 일자 자동생성
   - @LastModifiedBy : 데이터 수정자 자동생성
   - @LastModifiedDate : 데이터 수정 일자 자동생성

``` java
package com.ssg.homework.t2021hw.store.jpo.audit;

import lombok.Getter;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.Column;
import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.util.Date;


@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter
public class AuditJpo {

    @CreatedBy
    @Column(name = "REGPE_ID")
    private String regpeId; //등록자

    @CreatedDate
    @Column(name = "REG_DTS")
    private Date regDts;    //등록일시

    @LastModifiedBy
    @Column(name = "MODPE_ID")
    private String modpeId; //수정자

    @LastModifiedDate
    @Column(name = "MOD_DTS")
    private Date modDts;  //수정일시

    public void setModpeId(String modpeId) {
        this.modpeId = modpeId;
    }

    public void setRegpeId(String regpeId) {
        this.regpeId = regpeId;
    }
}
```

3. 생성한 Audit 객체를 상속받아 JPA 객체를 구현
``` java
@Table(name = "PAYMENT")
@Entity
@NoArgsConstructor
@Getter
@Setter
public class PaymentJpo extends AuditJpo {
```

4. @CreatedBy, @LastModifiedBy를 이용해 생성자, 수정자를 사용자 정의에 맞게 생성하려면 별도 작업이 필요.
 - AuditorAware	인터페이스를 Overriding하여 getCurrentAuditor 메소드를 실제 구현하면 구현된 값으로 생성자와 수정자 값이 생성할수있다.
 - 구현된 내용을 빈으로 등록해야한다.  

 &#8251; Http Header나 Security에서 정보를 받아서 저장할수도 있다.
``` java
package com.ssg.homework.t2021hw.config;

import org.springframework.data.domain.AuditorAware;

import java.util.Optional;

public class BasicAuditor implements AuditorAware<String> {
    @Override
    public Optional<String> getCurrentAuditor() {
        return Optional.of("payment-gateway");
    }
}
```
``` java
package com.ssg.homework.t2021hw.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.domain.AuditorAware;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
@Configuration
public class JpaConfiguration {

    @Bean
    public AuditorAware<String> auditorProvider() {
        return new BasicAuditor();
    }
}
```


### 3. JPA Auditing 테스트 결과

``` java
    @Test
    @DisplayName("audit 생성자/수정자 ID 테스트")
    public void auditingSaveTest() {
        PaymentDto paymentDto = new PaymentDto("0000000345", "P0002", null, 157400, "10");

        PaymentJpo result = paymentJpaRepository.save(new PaymentJpo(paymentDto));

        Assert.assertEquals(result.getModpeId(),"payment-gateway");
        Assert.assertEquals(result.getRegpeId(),"payment-gateway");

    }
```	
<img src = "https://github.com/people92/people92.github.io/blob/master/img/jpaAuditResult.png?raw=true">


___


## __참고__
https://web-km.tistory.com/42  
https://m.blog.naver.com/rorean/221485891788
