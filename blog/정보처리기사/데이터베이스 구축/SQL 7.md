태그: #정보처리기사 
연결문서: [SQL 8](SQL%208.md)


# 04. SQL 지원 도구

## 01. 데이터 사전(Data Dictionary)

### 1) 데이터 사전의 개념
- 데이터베이스에 포함되는 모든 객체에 관한 정의나 명세에 관한 정보를 메타 데이터 형태로 유지 관리하는 곳
    - 메타 데이터의 유형 : 사용자 정보(아이디, 패스워드 및 권한 등), 무결성 제약 정보, 함수, 트로시저 및 트리거 등
- 여러 가지 스키마와 이들 속에 포함된 사상들에 관한 정보도 컴파일되어 저장됨
- 사전 자체도 하나의 데이터베이스로 간주하며, 시스템 카탈로그라고도 함

<br>

### 2) 데이터 사전 테이블
- CHARACTER_SETS : 사용 가능한 모든 문자 셋에 대한 정보
- COLLATIONS : 데이터베이스에 저장된 값들을 비교, 검색하거나 정렬 등의 작업을 위해 문자들을 서로 비교할 때 사용하는 규칙들의 집합으로 사용 가능한 모든 콜레션에 대한 정보
- COLUMNS : 테이블 컬럼에 대한 콜레션 정보
- KEY_COLUMN_USAGE : 제약사항을 가지고 있는 키 컬럼에 대한 정보
- ROUTINES : Stored 루틴이란 데이터베이스 상에 저장된 SQL 구문으로 Stored 루틴에 대한 정보
- SCHEMATA : 하나의 스키마는 하나의 데이터베이스로, SCHEMATA는 데이터베이스의 정보
- TABLES : 데이터베이스에 존재하는 테이블에 대한 정보
- TABLE_CONSTRAINTS : 테이블의 제약사항에 대한 정보
- TRIGGERS : 트리거에 대한 정보
- VIEWS : 데이터베이스에 있는 뷰에 대한 정보 제공

<br>

---

<br>

## 02. 시스템 카탈로그(System Catalog)

### 1) 시스템 카탈로그의 개념
- 테이블 정보, 인덱스 정보, 뷰 정보 등을 저장하는 시스템 테이블
- 시스템 카탈로그는 사용자들이 검색은 가능하나 직접적인 변경은 불가능함
- 시스템 카탈로그는 데이터 정의문, 데이터 조작문이 기술되면 DBMS에 의해서 자동으로 갱신됨
- 시스템이 필요로 하는 모든 객체에 대한 정보를 가지고 있는 시스템 데이터베이스로 사용자 데이터베이스와는 구별됨
- 시스템 데이터베이스에는 카탈로그와 디렉터리로 나누어 저장하는데 카탈로그에는 객체에 대한 정보를 저장하고 디렉터리에는 그 정보를 어떻게 액세스할 것인가 하는 시스템만이 사용하는 정보를 저장함
- 관계형 시스템 카탈로그는 비 관계형 시스템 카탈로그에 비해 같은 인터페이스를 통해서 접근할 수 있으므로 일반 사용자도 그 내용을 검색할 수 있다는 장점을 가지고 있음

<br>

### 2) 시스템 카탈로그의 구성 요소
- 시스템 카탈로그 : 생성된 테이블에 관련된 정보를 기록함
- 데이터 디렉터리 : 데이터 사전에 수록된 데이터를 실제로 접근하는 데 필요한 정보를 기록하며 시스템만이 접근할 수 있는 구역
- 메타 데이터 : 사용자의 데이터를 설명해 놓은 데이터 사전
- 데이터 사전 : 사용자의 테이블과 속성의 관련 정보를 기록함

<br>

---

<br>

## 03. 응용 시스템의 DBMS 접속

### 1) 응용 시스템의 DBMS 접속 개념
- DBMS에서 사용되는 SQL은 데이터베이스 관리 도구를 사용하여 직접 접속해 실행할 수 있고, 응용 시스템에서도 호출 방식을 통해 사용이 가능함
- 사용자는 응용 시스템을 통해 DBMS에 접속하여 조회하고, 저장하는 것이 가장 일반적인 형태
- 응용 시스템은 입력 변수를 전달하고 SQL을 실행하여 결과를 응용 시스템을 통해 사용자에게 전달하는 일종의 매개체

<br>

### 2) JDBC(Java Database Connectivity)
- Java는 JDBC를 사용하여 데이터베이스에 연결함
- JDBC는 SQL을 사용하여 DBMS에 질의하고 데이터를 조작하는 API를 제공함
- JDBC에서 API를 사용하는 방법은 순수한 Java로 JDBC를 사용하는 방법과 동일함

<br>

### 3) MyBatis
- SQL Mapping 기반을 둔 오픈소스 접근 프레임워크
- DBMS에 질의하기 위한 SQL 쿼리를 별도의 XML 파일로 분리하고 Mapping을 통해서 SQL을 실행함
- MyBatis는 DBMS 의존도가 높고 SQL 질의 언어를 사용하기 때문에 SQL 친화적인 국내 실무 개발 환경에 많이 사용됨
- 복잡한 JDBC 코드를 단순화할 수 있음
- SQL을 거의 그대로 사용 가능함
- Spring 기반 프레임워크와 통합 기능을 제공함
- 우수한 성능을 보여줌

<br>

---

<br>

## 04. 특정 기능 수행 SQL

### 1) 응용 시스템 요구사항 정의
1. 전체 요구사항 취합
    - 응용 시스템을 통해서 구현해야 하는 서비스의 기능을 모두 정리함
    - 요구사항의 성격 및 기능, 상황에 맞게 분류하여 체계화함
2. 취합된 요구사항 분석
    - 응용 시스템에서 요구하는 사항 중 DBMS에 질의를 통해 구현 가능한 기능을 선별하고 기능을 분석함
    - 질의 유형(조회, 삽입, 수정, 삭제)을 파악함
    - 서비스의 가변 값을 파악함
    - 가변 값을 제외하고 달라질 수 있는 부분을 파악함

<br>

### 2) 구현 SQL 설계
1. 기능 분류를 통해 DQL, DML 또는 프로시저 중 어느 것을 사용할 것인지를 결정함
    - 질의 유형으로부터 필요한 SQL을 정의함
    - 프로시저를 작성해야 할지를 판단함
    - 사용자 함수를 작성해야 할지를 판단함
    - MERGE를 잘 활용함
2. 구현을 위한 구체적 SQL을 설계함
    - 단순한 구조를 유지할 수 있도록 설계함
    - 관련된 테이블을 확인함
    - 부모-자식 관계 간의 테이블에 삽입하는 경우에는 순서를 확인함
    - 데이터를 확인한
    - 입력 변수를 확인함

<br>

---

<br>

## 05. SQL 지원 도구

### 1) PL/SQL 도구
1. 프로그래밍 언어의 특성을 수용한 SQL의 확장 기능
2. 컴파일이 필요 없이 스크립트 형태로 작성한 후 실행이 가능함
3. 모듈화가 가능함
    - 블록 내에서 문장(명령어들의 집합)들을 묶을 수 있음
    - 적절히 나눈 모듈들은 복잡한 문제를 쉽게 해결할 수 있게 함
    - 작은 블록들을 큰 블록에 포함하여 강력한 프로그램을 작성함
    - 블록 내에서 논리적으로 그룹화가 가능함
4. 식별자를 선언할 수 있음
    - 변수, 상수 등을 선언하여 사용할 수 있음
    - 테이블과 레코드 형태의 동적 변수를 선언할 수 있음
    - 단일형이나 복합형 데이터 타입을 선언할 수 있음
5. 절차형 언어 구조로 된 프로그램을 작성할 수 있음
    - IF 문을 사용하여 명령을 선택할 수 있음
    - LOOP 문을 사용하여 명령을 반복할 수 있음
    - EXPLICIT CURSOR를 사용하여 튜플을 다중 처리할 수 있음
6. ERROR 처리가 가능함
    - EXCEPTION 처리 루틴을 사용하여 오류를 처리할 수 있음
    - 사용자 오류를 EXCEPTION 처리 루틴으로 처리할 수 있음
7. 성능 향상을 기대할 수 있음
    - PL/SQL은 부하를 감소시켜 성능을 향상시킬 수 있음
    - PL/SQL은 SQL 명령어들을 블록으로 묶어 전송할 수 있기 때문에 성능을 향상시킬 수 있음

<br>

### 2) SQL*Plus 도구
1. SQL을 DBMS 서버에 전송하여 처리할 수 있도록 하는 Oracle에서 제공하는 도구
2. SQL과 SQL*Plus 비교
    - SQL
        - 데이터베이스와 통신하는 언어
        - 데이터와 테이블에 대한 정의가 가능함
        - 여러 행 입력이 가능함
        - SQL 버퍼를 사용함
        - 키워드를 축약할 수 없음
        - 종료 문자(;)를 사용함
        - ANSI 표준에 기초함
    - SQL*Plus
        - SQL 문장을 서버에 전송하는 도구
        - 데이터에 대한 어떤 정의도 불가능함
        - 여러 행 입력이 불가능함
        - SQL 버퍼를 사용하지 않음
        - 키워드를 축약할 수 있음
        - 종료 문자(;)를 사용하지 않음
        - Oracle에서 제공하는 도구

<br>

### 3) APM(Application Performance Management)
1. 운영 중인 시스템에 대한 가용성 확보, 다운 타임 최소화 등을 통해 안정적인 시스템 운영을 위하여, 부하량과 접속자 파악 및 장애 진단 등을 목적으로 하는 성능 모니터링 도구
2. APM 유형에는 리소스와 엔드투엔드 모니터링이 있음
3. 자원(Resource) 모니터링
    - 모니터링 대상 자원은 CPU, 메모리, 네트워크, 디스크 등이 있음
    - 대표적인 오픈소스로는 Nagios, Zabbix, Cacti 등이 있음
4. End to End 모니터링
    - 모니터링 대상을 응용 프로그램 실행 관점으로 비즈니스 트랜잭션 관리 및 최종 사용자 등 End to End 모니터링을 봄
    - 대표적인 오픈소스로는 VisualVM이 있고, 상용 제품으로는 제니퍼, 파로스, 시스 마스터 등이 있음

<br>

### 4) TKPROF 도구
1. 실행되는 SQL 문장에 대해 분석 정보를 제공하여 개발자가 특정 SQL 문장을 어떻게 사용해야 할 것인지에 대한 가이드라인을 제공해 주는 도구
2. TKPROF는 EXPLAIN PLAN과 병행하여 사용하는 것이 좋음
3. TKPROF의 추적 결과로 파악할 수 있는 분석 정보 내용
    - Parse, Execute, Fetch 수
    - CPU 시간/경과된 시간
    - 물리적/논리적 Reads
    - 처리된 튜플 수
    - 라이브러리 캐시 Misses
    - 파싱이 발생할 때의 사용자
    - Commit/Rollback
4. Trace 유형
    - Instance Level 추적 : 지속적인 설정 방법으로 모든 SQL 수행에 대한 Trace 파일을 생성하므로 많은 부하가 발생함
    - Session Level 추적 : 임시적인 설정 방법으로 특정 프로세스별로 추적 파일을 생성함
5. Trace 유형 활용
    - Instance Level로 모든 SQL을 Trace하는 경우는 거의 없고, 데이터베이스 응용 프로그램에서 사용되는 특정 SQL에 대해서만 Trace하는 Session Level을 일반적으로 활용함
    - Trace를 Enable하는 것을 DB 부하가 수반되므로, 필요할 때만 사용하고 평상시 개발 및 운영 환경에서는 Disable하는 것이 좋음

<br>

### 5) EXPLAIN PLAN 도구
1. 사용자들이 SQL문의 액세스 경로를 확인하여 성능 개선을 할 수 있도록 SQL문을 분석하고 해석하여 실행 계획을 수립하고, 관련 테이블에 저장하도록 지원해주는 도구
2. EXPLAIN PLAN 결과 항목

```sql
SQL> set autotrace on /* DB에 접속하여 autotrace on으로 설정*/

SQL> SELECT * FROM 테이블명 /* 임의 SQL 명령을 실행하면 결과를 출력한 후에 EXPLAIN PLAIN의 결과 항목과 함께 실제 처리된 Statistics(통계) 값을 출력해줌*/
```

<br>

### 6) 소스 코드 인스펙션 도구(Source Code Inspection)
1. 데이터베이스의 성능 향상을 위해서 데이터 조작 프로시저 코드를 보면서 성능의 문제점을 개선해 나가는 활동
2. USER_SOURCE 데이터 사전을 활용
    - USER_SOURCE 데이터 사전으로 개발된 모든 데이터 조작 프로시저를 확인할 수 있음

```sql
SQL> SELECT DISTINCT(name) FROM USER_SOURCE
    WHERE TYPE = 'PROCEDURE';
```

3. 프로시저 소스 확인
    - 데이터 조작 프로시저는 USER_SOURCE의 text 조회를 통해서 확인할 수 있음

```sql
SQL> SELECT text FROM USER_SOURCE
    WHERE name = 'FORCURSOR_TEST';
```

<br>

### 7) SQL 성능 측정
- 데이터 조작 프로시저 최적화는 데이터베이스의 프로시저를 효율적으로 수정하여 성능을 개선하는 활동
- 프로시저 SQL의 실행 명령을 분석 후 효율적이지 않은 부분을 수정하여 최적의 실행 결과가 나올 수 있도록 함
- 쿼리 성능을 측정하고 개선하는 도구에는 TKPROF, EXPLAIN PLAN, 소스 코드 인스펙션 등이 있음
- 개발자는 SQL 특성을 파악하여 SQL문을 적절히 구사할 수 있는 능력을 기본적으로 갖추어야 함
- 개발자가 SQL 작성 시, 최적화 프로그램이 실행 계획을 수립한 후 실행되는 일련의 과정을 이해하고 작성하여야 함
- 구문 분석 단계 시 최적화 프로그램이 수립한 실행 계획에 따라 엄청난 수행 속도 차이가 발생할 수 있음을 이해해야 함
- 특정 SQL이 실행될 때 최적화 프로그램에 의해 수립된 실행 계획은 제어하기가 어려움
- 최적화 프로그램이 비정상적으로 동작한다면, 이를 추적하여 개발자가 원하는 실행계획으로 동작될 수 있도록 조정하는 과정이 필요함
- 개발자는 최적화 프로그램이 정상적인 실행 계획을 수립할 수 있도록 종합적이고 전략적인 포인트를 SQL에 부여하여 작성하여야 함
- 좋은 SQL은 추출되는 결과를 추론하여 SQL을 집합적으로 접근하여 작성하여야 함