# Querydsl 세팅

## build.gradle - dependencies

    //Querydsl 추가
    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
    annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jakarta"
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"

## Q타입 생성법
- Gradle -> Tasks -> build -> clean
- Gradle -> Tasks -> other -> compileQuerydsl

## Q타입 생성 확인
- build -> generated -> sources -> annotationProcessor ...   

## H2 DB 다운 및 설치
- [https://www.h2database.com]()
- 스프링 부트 3.x를 사용하면 2.1.214 버전 이상 사용
- 데이터베이스 파일 생성 방법
```shell
 jdbc:h2:~/querydsl (최소 한번)
 
~/querydsl.mv.db 파일 생성 확인

이후 부터는 이렇게 접속 jdbc:h2:tcp://localhost/~/querydsl
```

## application.yml 설정
- show_sql : 옵션은 System.out에 하이버네이트 실행 SQL을 남긴다.
- org.hibernate.SQL: 옵션은 logger를 통해 하이버네이트 실행 SQL을 남긴다.

# Querydsl 문법

## 기본 문법
- JPAQueryFactory를 생성할 때 제공하는 EntityManager에 여러 쓰래드가 동시에 접근해도 트랜잭션 마다 별도의 영속성 컨텍스트를 제공하여 동시성 문제 없음.


- Q클래스 인스턴스를 사용하는 2가지 방법
    

    QMember qMember = new QMember("m"); //별칭 직접 지정
    QMember qMember = QMember.member; // 기본 인스턴스 사용

기본 인스턴스를 static import와 함께 사용
import static study.querydsl.entity.QMember.*;

## JPQL이 제공하는 모든 검색 조건 제공
    member.username.eq("member1") // username = 'member1'
    member.username.ne("member1") // username != 'member1'
    member.username.eq("member1").not() // username != 'member1'
    
    member.username.isNotNull()
    
    member.age.in(10, 20) // age in (10, 20)
    member.age.notIn(10, 20) // age not in (10, 20)
    member.age.between(10, 20) // age between 10 and 20

    member.age.goe(30) // age >= 30
    member.age.gt(30) // age > 30
    member.age.loe(30) // age <= 30
    member.age.lt(30) // age < 30

    member.username.like("member%") //like 검색
    member.username.contains("member") // like '%member%'
    member.username.startsWith("member") // like 'member%'