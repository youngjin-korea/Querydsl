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