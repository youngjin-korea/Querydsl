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

## AND 조건 파라미터로 처리 
- where() 에 파라미터로 검색조건을 추가하면 AND 조건이 추가됨
- 이 경우 `null` 값은 무시됨 -> 동적쿼리 깔끔하게 사용가능  


    queryFactory
      .selectFrom(member)
      .where(member.username.eq("member1"), member.age.eq(10)).fetch();

## 결과 조회 
- `fetch()` : 리스트 조회, 리스트 없으면 빈 리스트 반환
- `fetchOne()` : 단 건 조회
  - 결과 없으면 `null`
  - 결과가 둘 이상이면 : com.querydsl.core.NonUniquerResultException
- `fetchFirst()` : limit(1).fetchOne() 
- `fetchResults()` : 페이징 정보 포함, total count 쿼리 추가 실행
- `fetchCount()` : count 쿼리로 변경해서 count 수 조회

## 정렬
- desc(), asc() : 일반정렬
- nullsLast(), nullsFirst() : null 데이터 순서 부여


    // 나이 내림차순(desc), 이름 올림차순(asc), 2에서 회원 이름이 없으면 마지막 출력(nulls last)
    List<Member> result = queryFactory
      .selectFrom(member)
      . where(member.age.eq(100))
      .orderBy(member.age.desc(), member.username.asc().nullsLast())
      .fetch();

## 페이징
- 조회 건수 제한


    // offset(3).limit(2)를 호출하면?
    // Offset(3): 0, 1, 2번 인덱스까지 총 3개를 건너뜁니다.
    // Limit(2): 그 다음 순서인 3번 인덱스부터 딱 2개를 가져옵니다.
    List<Member> result = queryFactory
      .selectFrom(member)
      .orderBy(member.username.desc())
      .offset(1) // 스킵할 개수: 1 -> 0번 인덱스 1개 스킵
      .limit(2) // 가져올 최대 데이터 수 : 2개
      .fetchResults();
- 전체 조회수


    // count 쿼리에 조인이 필요없는 성능 최적화가 필요하다면, count 전용 쿼리를 별 도로 작성해야 한다
    QueryResult<Member> queryResults = queryFactory
      .selectFrom(member)
      .orderBy(member.username.desc())
      .offset(1)
      .limit(2)
      .fetchResults(); // count 쿼리가 추가로 실행됨 

## 집합
- sum(), count(), avg(), max(), min()

## 서브쿼리
- JPAExpressions 사용
- inline 뷰 지원 안됨 (from절 서브쿼이 x)

    
    QMember memberSub = new QMember("memberSub") 
    List<Member> result = queryFactory
      .selectFrom(member)
      .where(member.age.eq(
          JPAExpressions
            .select(memberSub.age.max())
            .from(memberSub)
          ))
          .fetch();

## Case 문
- select, where, order by절에서 가능
- 단순 : member.age.when(10).then("열살").otherwise("기타")
- 복잡 : new CaseBuilder().when(member.age.between(0, 20)).then("0~20살").otherwise("기타")
- 