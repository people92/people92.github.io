---
layout: post
title:  "Spring Data Envers"
date:   2021-06-08
excerpt: "Spring Data Envers"
tag:
- markdown 
comments: false
---

# Spring Data Envers  

## 목차
1. Spring Data Envers


___

## __개요__

변경이력을 관리해야하는데 당연히 이력테이블을 새로 만들어 진행하려고 했는데,
구글링 중 우연히 Spring Envers 발견
이력테이블을 추가하려고 생각하고 있었는데 Spring Data Envers라는 변경이력을 관리해주는 라이브러리가 존재하였다.
하지만, 사용할 수 있을지는 모르겠다(테이블명, 항목명 표준 이슈)


___

## __상세__

### 1. 하이버네이트 Envers
하이버네이트 핵심 모듈  
JPA스펙에 정의된 모든 매핑 관리  
엔티티의 변경이력을 자동으로 관리  

XX Table -> XXX_AUD 테이블로 이력관리  
히스토리를 계속 쌓는 방식으로 관리  
REV == 리비전 식별자  

REVTYPE  
0 : 등록, 1 : 수정, 2 : 삭제  


@Audited : 히스토리 관리(클래스 or 필드))  
@NotAudited : 히스토리 관리 제외(필드)  

revision 관리  
- REVINFO 테이블 제공  
- 트랜잭션 단위의 통합 Revision(개정) 키 관리  
- Revision 생성 시간 관리  
- 사용자 정의 필드 추가 기능  

envers 조회  

- 필드변경여부관리  
@Audited(withModifiedFlag = true)  
AUD테이블에 필드마다 수정 상태 컬럼 추가  
ex)NAME -> NAME_MOD  
어떤 필드를 수정했는지 알수 있다.  
필드의 수정여부를 검색조건으로 사용 가능  

- 같은 트랜잭션에서 함께 변경된 엔티티를 검색  
  org.hibernate.envers.track_entities_changed_in_revision: true  
  REVCHANGES 테이블 생성  

기타  
@Audited(targetAuditMode = NOT_AUDITED) : 연관관계가 있는 엔티티를 감사하지 않음  
@AuditOverride : 상속관계  


Spring data envers  
- 스프링 데이터 JPA의 확장 모듈  
- 하이버네이트 Envers를 편리하게 조회하도록 도움  
- RevisionRepository 인터페이스  
- 편리한 메타데이터 조회  
spring-data-envers 디팬던시 추가  
@EnableJpaRepositories(repositoryFactoryBeanClass = EnversRevisionRepositoryFactoryBean.class)  

스프링 데이터 엔버스 단점  
: 복잡한 조회 불가, 버전업 잘 안되는편 , QueryDSL 관련 기능과 함께 사용하려면 코드 수정 해야함  

메인테이블 변경시 이력테이블도 alter 해야함  

___

## __참고__
