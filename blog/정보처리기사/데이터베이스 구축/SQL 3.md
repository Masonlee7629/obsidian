태그: #정보처리기사
연결문서: [SQL 4](SQL%204.md)

# 01. SQL 명령어

## 05. 데이터 제어어(DCL)

### 1) DCL과 TCL
1. DCL의 조작 대상, 오브젝트 유형
    - 사용자 권한 : 접근 통제가 목적이며, 사용자를 등록하고, 사용자에게 특정 데이터베이스를 사용할 수 있는 권리를 부여하는 작업
    - 트랜잭션 : 안전한 거래 보장이 목적이며, 동시에 다수의 작업을 독립적으로 안전하게 처리하기 위한 상호 작용 단위
    - 트랜잭션 제어를 위한 명령어 TCL이 있음
    - TCL과 DCL은 대상이 달라 서로 별개의 개념으로 분류할 수 있으나, 제어 기능의 공통점으로 DCL의 일부로 분류하기도 함
2. DCL과 TCL 명령어
    - DCL(Data Control Language)
        - GRANT : 데이터베이스 사용자에게 권한을 부여함
        - REVOKE : 데이터베이스 사용자에게 권한을 회수함
    - TCL(Transaction Control Language)
        - COMMIT : 트랜잭션을 확정함
        - ROLLBACK : 트랜잭션을 취소함
        - CHECKPOINT : 복귀 지점을 설정함

<br>

### 2) GRANT 명령어

#### 1. GRANT 명령어 형식

```sql
GRANT [ 부여할 시스템 권한명|ROLE명 ] TO
    [ 사용자|ROLE명|PUBLIC ]
    [ WITH GRANT/ADMIN OPTION ];
```

- PUBLIC : 시스템 권한이나 데이터베이스 권한을 모든 사용자에게 부여함
- WITH GRANT OPTION : 권한을 부여 받은 사용자가 또 다른 사용자에게 권한을 부여하거나 수정, 회수할 수 있는 권한을 부여하며, 이 권한을 갖는 사용자들이 체인 형식으로 증가할 수 있음
- WITH ADMIN OPTION : 권한을 부여 받은 사용자가 또 다른 사용자에게 권한을 부여하지만 회수할 수 없는 권한을 부여함

#### 2. 시스템 권한 형식

```sql
GRANT 권한1, 권한2 TO 사용자계정;
```

- CREATE USER : 계정 생성 권한
- DROP USER : 계정 삭제 권한
- DROP ANY TABLE : 테이블 삭제 권한
- CREATE SESSION : 데이터베이스 접속 권한
- CREATE TABLE : 테이블 생성 권한
- CREATE VIEW : 뷰 생성 권한
- CREATE SEQUENCE : 시퀀스 생성 권한
    - 데이터 삽입 시 자동으로 증가되는 숫자의 시작점이나 증가폭을 지정할 수 있는 권한
- CREATE PROCEDURE : 함수 생성 권한
    - 자주 사용하는 SQL 명령을 함수나 프로시저로 작성할 수 있는 권한

#### 3. 객체 권한 형식

```sql
GRANT 권한1, 권한2 ON 객체명 TO 사용자계정;
```

- ALTER : 테이블 구조 변경 권한
- INSERT : 데이터 삽입 권한
- DELETE : 데이터 삭제 권한
- SELECT : 데이터 조회 권한
- UPDATE : 데이터 갱신 권한
- EXECUTE ON PROCEDURE : 실행 권한

<br>

### 3) 사용자 그룹 관리의 개념
- 데이터베이스에 접속하여 사용하는 대상자를 사용자라고 하며, 다수의 사용자가 접속하는 사용자를 사용자 그룹이라고 함
- 회사 내부나 외부에서 데이터를 공유하기 위해 데이터베이스를 오픈하게 됨
    - 데이터베이스 전체를 오픈하지 않고 필요한 부분만 제한적으로 오픈함
- 사용자 그룹은 동일한 권한과 제약을 가지는 사용자들이 공통으로 사용하는 계정
- 사용자 그룹의 관리는 역할 기반 접근 제어(RBAC : Role Based Access Control) 그룹 관리를 기반으로 함
- 사용자를 개별적으로 분할하지 않고 수행하는 역할을 기반으로 그룹화하여 먼저 분할하고 사용자 그룹에 권한을 부여함
    - 개별 사용자를 사용자 그룹에 맞추어 배정하고 결국 사용자는 사용자 그룹에 속한 권한을 사용하게 함

<br>

### 4) 역할(ROLE) 부여를 통한 사용자 그룹 관리
- 역할(ROLE)은 데이터베이스에서 사용자 그룹과 권한 사이에서 중개 역할을 하며, 이를 통해 빠르고 정확하게 필요한 권한을 부여할 수 있게 함
- DQL, DML, DDL 모두 권한 부여가 가능하며 각각 분리하여 권한을 부여하는 것이 일반적
- DQL과 DML에 제한적 권한을 부여하더라도 DDL 권한이 있는 경우 DDL을 통해 생성한 테이블을 생성하거나 삭제할 수 있기 때문에 주의해야 함
- 일반 사용자 권한 개념으로 DQL, 중간 관리자 권한으로 DML까지 부여하고 데이터베이스 관리자 권한 개념으로 DDL을 부여하는 방법을 사용

<br>

### 5) 사용자 그룹 변경
- 생성해 놓은 사용자 그룹을 삭제하거나 패스워드를 변경할 수 있음

```sql
ALTER USER [사용자 그룹 ID] IDENTIFIED BY [패스워드];
```

<br>

### 6) ROLE 명령어

#### 1. ROLE 명령어 형식 - 권한 부여 집합

```sql
CREATE ROLE ROLE명;
```

#### 2. ROLE 사용 방법
- 시스템 권한을 부여하고, 회수할 때와 동일한 명령으로 사용자에게 권한 부여와 회수를 쉽게 할 수 있음
- ROLE은 CREATE ROLE 권한을 가진 사용자에 의해서 생성됨
- 한 사용자가 여러 개의 ROLE을 접근할 수 있고, 여러 사용자에게 같은 ROLE을 부여할 수 있음
- ROLE은 또 다른 사용자에게 ROLE을 부여할 수 있음
- DBA가 권한을 부여할 때 사용자별로 부여 한다면 복잡하고 관리가 어려움
- DBA가 사용자 역할에 맞게 ROLE을 생성해서 ROLE만 사용자에게 부여하면 효율적으로 사용자들의 권한을 관리할 수 있음

#### 3. ROLE 사용 예시
- member_right이란 ROLE명을 생성함

```sql
CREATE ROLE member_right;
```

- 테이블 생성, 뷰 테이블 생성 권한을 member_right에 부여함

```sql
GRANT create table, create view TO member_right;
```

- member_right에 부여된 ROLE을 사용자 andy, sherri에게 부여함

```sql
GRANT member_right TO andy, sherri;
```

<br>

### 7) REVOKE 명령어

#### 1. REVOKE 명령어 형식
- 사용자에게 권한을 회수하는 명령어
- 사용자에게 부여된 권한을 다시 회수할 수 있음
    - 권한을 부여하는 것과 반대 개념이며, 사용하는 방법도 유사함
- REVOKE를 사용하여 회수며, TO가 아닌 FROM으로 사용자 그룹은 선택함

```sql
REVOKE [ 회수할 시스템 권한명|ROLE명 ] FROM
    [ 사용자계정|ROLE명|PUBLIC ];
```

#### 2. 시스템 권한 회수 형식

```sql
REVOKE 권한1, 권한2 FROM 사용자계정;
```

#### 3. 객체 권한 회수 형식

```sql
REVOKE 권한1, 권한2 ON 객체명 FROM 사용자계정;
```

#### 4. REVOKE 사용 예시
- andy에게 부여한 생성, 수정, 삭제 권한을 회수함

```sql
REVOKE CREATE USER, ALTER USER, DROP USER FROM andy;
```