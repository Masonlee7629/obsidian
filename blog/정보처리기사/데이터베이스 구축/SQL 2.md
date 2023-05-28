태그: #정보처리기사 
연결문서: [SQL 3](SQL%203.md)

# 01. SQL 명령어

## 03. 데이터 조작어(DML)

### 1) DML 명령어 종류
- 데이터 삽입(INSERT) : 삽입 형태로 신규 데이터를 테이블에 저장함
- 데이터 수정(UPDATE) : 테이블의 내용을 수정함
- 데이터 삭제(DELETE) : 테이블의 내용을 삭제함
- 데이터 조회(SELECT) : 테이블의 내용을 조회함

<br>

### 2) 데이터 삽입
- 입력하고자 하는 테이블에 모든 컬럼 데이터를 입력한다면 컬럼명을 명시하지 않아도 되지만, 특정 컬럼만을 입력하고자 한다면 반드시 컬럼명을 명시해야 함
- 컬럼명 수와 VALUES 절의 수는 반드시 동일해야 함
- 기존에 존재하는 테이블 데이터로부터 특정 테이블로 데이터를 복사할 수 있음
<br>

1. 필드명을 지정하는 경우

```sql
INSERT INTO 테이블명(필드명1, 필드명2, 필드명3, ...)
    VALUES (값1, 값2, 값3, ...);
```

2. 필드명을 지정하지 않는 경우

```sql
INSERT INTO 테이블명 VALUES (값1, 값2, 값3, ...);
```

3. 특정 테이블로 복사하는 경우

```sql
INSERT INTO 사본테이블명(필드명1, 필드명2, ...)
    SELECT 필드명1, 필드명2, ... FROM 원본테이블명;
```

<br>

### 3) 테이블 수정(UPDATE)
- UPDATE 명령문은 보통 WHERE 절을 통해 어떤 조건이 만족할 경우에만 특정 필드의 값을 수정하는 용도로 많이 사용됨
- 값에는 상수뿐만 아니라 수식도 사용이 가능함

```sql
UPDATE 테이블명 SET 필드1 = 값1, 필드명2 = 값2, ...
    [WHERE 절];
```

<br>

### 4) 데이터 삭제(DELETE)
- 튜플을 선택적으로 삭제할 때 사용됨
- 조건절 없이 DELETE를 사용하는 경우, 테이블 전체가 한 번에 삭제되는 위험이 있음

```sql
DELETE FROM 테이블명 [WHERE 절];
```

<br>

### 5) 데이터 조회(SELECT)
- 데이터의 내용을 조회할 때 사용하는 명령어
- 가장 많이 사용되는 SQL 명령어

```sql
SELECT [ OPTION ] 필드명 FROM 테이블명 [WHERE 절];
```

<br>

---

<br>

## 04. SELECT 명령어

### 1) SELECT 명령어 형식
1. SELECT 명령어 형식

```sql
SELECT [ DISTINCT ] {*|필드명1, ...|필드명1 AS 별명1, ...|연산}
    FROM 테이블명
    { WHERE 조건 | GROUP BY 필드명 [ HAVING 조건 ]}
    [ ORDER BY { 정렬 필드명 [ ASC|DESC ]}];
```

2. SELECT 명령어의 옵션(Option)
    - DISTINCT : 중복되는 튜플(레코드, 행)을 제거하여 조회함
    - * : 모든 필드를 조회함
    - 필드명1 : 필드명1, 필드명2, ... 으로 여러 개를 지정할 수 있음
    - 별명1 : 필드명이나 연산 결과를 별명으로 제목을 부여할 수 있음
    - 연산 : 사칙연산, SUM(), AVG() 등의 함수를 사용할 수 있음
    - WHERE 조건 : 조건을 만족하는 튜플들만 조회함
    - GROUP BY 필드명 : 그룹별로 묶을 필드명을 지정함
    - HAVING : 그룹별로 조건을 만족하는 튜플들만 조회함
    - ORDER BY : 정렬 옵션으로 ASC(오름차순, 생략 가능), DESC(내림차순), 1차키 정렬 필드, 2차 정렬 필드, ...를 지정할 수 있음

<br>

### 2) SELECT의 기본적인 명령
<테이블명 : 학생>

| &nbsp;&nbsp;학번&nbsp;&nbsp; | &nbsp;&nbsp;이름&nbsp;&nbsp; | &nbsp;&nbsp;학과코드&nbsp;&nbsp; | &nbsp;&nbsp;시험점수&nbsp;&nbsp; | &nbsp;&nbsp;과제&nbsp;&nbsp; | &nbsp;&nbsp;학점&nbsp;&nbsp; |
|:-:|:-:|:-:|:-:|:-:|:-:|
| &nbsp;&nbsp;H01&nbsp;&nbsp; | &nbsp;&nbsp;최성호&nbsp;&nbsp; | &nbsp;&nbsp;A01&nbsp;&nbsp; | &nbsp;&nbsp;98&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;A&nbsp;&nbsp; |
| &nbsp;&nbsp;H02&nbsp;&nbsp; | &nbsp;&nbsp;김희정&nbsp;&nbsp; | &nbsp;&nbsp;A02&nbsp;&nbsp; | &nbsp;&nbsp;47&nbsp;&nbsp; | &nbsp;&nbsp;0&nbsp;&nbsp; | &nbsp;&nbsp;F&nbsp;&nbsp; |
| &nbsp;&nbsp;H03&nbsp;&nbsp; | &nbsp;&nbsp;이재춘&nbsp;&nbsp; | &nbsp;&nbsp;A01&nbsp;&nbsp; | &nbsp;&nbsp;86&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;B&nbsp;&nbsp; |
| &nbsp;&nbsp;H04&nbsp;&nbsp; | &nbsp;&nbsp;임승현&nbsp;&nbsp; | &nbsp;&nbsp;E01&nbsp;&nbsp; | &nbsp;&nbsp;28&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;NULL&nbsp;&nbsp; |
| &nbsp;&nbsp;H05&nbsp;&nbsp; | &nbsp;&nbsp;김현태&nbsp;&nbsp; | &nbsp;&nbsp;C07&nbsp;&nbsp; | &nbsp;&nbsp;67&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;D&nbsp;&nbsp; |
| &nbsp;&nbsp;H06&nbsp;&nbsp; | &nbsp;&nbsp;김유신&nbsp;&nbsp; | &nbsp;&nbsp;C07&nbsp;&nbsp; | &nbsp;&nbsp;98&nbsp;&nbsp; | &nbsp;&nbsp;0&nbsp;&nbsp; | &nbsp;&nbsp;A&nbsp;&nbsp; |
| &nbsp;&nbsp;H07&nbsp;&nbsp; | &nbsp;&nbsp;정용조&nbsp;&nbsp; | &nbsp;&nbsp;A01&nbsp;&nbsp; | &nbsp;&nbsp;37&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;F&nbsp;&nbsp; |
| &nbsp;&nbsp;H08&nbsp;&nbsp; | &nbsp;&nbsp;신사임&nbsp;&nbsp; | &nbsp;&nbsp;D03&nbsp;&nbsp; | &nbsp;&nbsp;95&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;A&nbsp;&nbsp; |
| &nbsp;&nbsp;H09&nbsp;&nbsp; | &nbsp;&nbsp;김유신&nbsp;&nbsp; | &nbsp;&nbsp;E01&nbsp;&nbsp; | &nbsp;&nbsp;69&nbsp;&nbsp; | &nbsp;&nbsp;0&nbsp;&nbsp; | &nbsp;&nbsp;D&nbsp;&nbsp; |
| &nbsp;&nbsp;H10&nbsp;&nbsp; | &nbsp;&nbsp;윤정연&nbsp;&nbsp; | &nbsp;&nbsp;E01&nbsp;&nbsp; | &nbsp;&nbsp;89&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;B&nbsp;&nbsp; |

<br>

1. <학생> 테이블의 모든 내용을 검색(조회)함

```sql
SELECT * FROM 학생;
```

2. 이름과 시험점수만을 검색함

```sql
SELECT 이름, 시험점수 FROM 학생;
```

3. 중복된 이름을 제거하여 검색함

```sql
SELECT DISTINCT 이름 FROM 학생;
```

4. 시험점수에 5를 빼서 출력함

```sql
SELECT 시험점수 - 5 FROM 학생;
```

5. 학과코드가 중복된 것은 제외하고 그 개수를 출력함

```sql
SELECT COUNT(DISTINCT 학과코드) FROM 학생;
```

6. 학과코드가 중복된 것은 제외하고 학과코드가 A01인 개수를 출력함

```sql
SELECT COUNT(DISTINCT 학과코드) FROM 학생
    WHERE 학과코드 = 'A01';
```

7. 시험 점수 합계를 출력함

```sql
SELECT SUM(시험점수) FROM 학생;
```

8. 시험점수의 평균을 출력함(단, NULL 학점은 제외함)

```sql
SELECT SVG(시험점수) FROM 학생
    WHERE 학점 IS NOT NULL;
```

9. 시험점수의 최댓값을 출력함

```sql
SELECT MAX(시험점수) FROM 학생;
```

10. 시험점수의 최솟값을 출력함

```sql
SELECT MIN(시험점수) FROM 학생;
```

<br>

### 3) SELECT의 조건 지정 검색 명령
<테이블명 : 학생>

| &nbsp;&nbsp;학번&nbsp;&nbsp; | &nbsp;&nbsp;이름&nbsp;&nbsp; | &nbsp;&nbsp;학과코드&nbsp;&nbsp; | &nbsp;&nbsp;시험점수&nbsp;&nbsp; | &nbsp;&nbsp;과제&nbsp;&nbsp; | &nbsp;&nbsp;학점&nbsp;&nbsp; |
|:-:|:-:|:-:|:-:|:-:|:-:|
| &nbsp;&nbsp;H01&nbsp;&nbsp; | &nbsp;&nbsp;최성호&nbsp;&nbsp; | &nbsp;&nbsp;A01&nbsp;&nbsp; | &nbsp;&nbsp;98&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;A&nbsp;&nbsp; |
| &nbsp;&nbsp;H02&nbsp;&nbsp; | &nbsp;&nbsp;김희정&nbsp;&nbsp; | &nbsp;&nbsp;A02&nbsp;&nbsp; | &nbsp;&nbsp;47&nbsp;&nbsp; | &nbsp;&nbsp;0&nbsp;&nbsp; | &nbsp;&nbsp;F&nbsp;&nbsp; |
| &nbsp;&nbsp;H03&nbsp;&nbsp; | &nbsp;&nbsp;이재춘&nbsp;&nbsp; | &nbsp;&nbsp;A01&nbsp;&nbsp; | &nbsp;&nbsp;86&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;B&nbsp;&nbsp; |
| &nbsp;&nbsp;H04&nbsp;&nbsp; | &nbsp;&nbsp;임승현&nbsp;&nbsp; | &nbsp;&nbsp;E01&nbsp;&nbsp; | &nbsp;&nbsp;28&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;NULL&nbsp;&nbsp; |
| &nbsp;&nbsp;H05&nbsp;&nbsp; | &nbsp;&nbsp;김현태&nbsp;&nbsp; | &nbsp;&nbsp;C07&nbsp;&nbsp; | &nbsp;&nbsp;67&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;D&nbsp;&nbsp; |
| &nbsp;&nbsp;H06&nbsp;&nbsp; | &nbsp;&nbsp;김유신&nbsp;&nbsp; | &nbsp;&nbsp;C07&nbsp;&nbsp; | &nbsp;&nbsp;98&nbsp;&nbsp; | &nbsp;&nbsp;0&nbsp;&nbsp; | &nbsp;&nbsp;A&nbsp;&nbsp; |
| &nbsp;&nbsp;H07&nbsp;&nbsp; | &nbsp;&nbsp;정용조&nbsp;&nbsp; | &nbsp;&nbsp;A01&nbsp;&nbsp; | &nbsp;&nbsp;37&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;F&nbsp;&nbsp; |
| &nbsp;&nbsp;H08&nbsp;&nbsp; | &nbsp;&nbsp;신사임&nbsp;&nbsp; | &nbsp;&nbsp;D03&nbsp;&nbsp; | &nbsp;&nbsp;95&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;A&nbsp;&nbsp; |
| &nbsp;&nbsp;H09&nbsp;&nbsp; | &nbsp;&nbsp;김유신&nbsp;&nbsp; | &nbsp;&nbsp;E01&nbsp;&nbsp; | &nbsp;&nbsp;69&nbsp;&nbsp; | &nbsp;&nbsp;0&nbsp;&nbsp; | &nbsp;&nbsp;D&nbsp;&nbsp; |
| &nbsp;&nbsp;H10&nbsp;&nbsp; | &nbsp;&nbsp;윤정연&nbsp;&nbsp; | &nbsp;&nbsp;E01&nbsp;&nbsp; | &nbsp;&nbsp;89&nbsp;&nbsp; | &nbsp;&nbsp;1&nbsp;&nbsp; | &nbsp;&nbsp;B&nbsp;&nbsp; |

<br>

1. 시험점수가 70점 이상인 학생의 이름, 시험점수, 학점을 검색함

```sql
SELECT 이름, 시험점수, 학점 FROM 학생
    WHERE 시험점수 >= 70;
```

2. 시험점수가 60점 이상이고 90점 미만인 학생 이름을 검색함

```sql
SELECT 이름 FROM 학생
    WHERE 시험점수 >= 60 AND 시험점수 < 90;
```

3. 시험점수가 60점 이상이거나 과제가 1인 학생의 이름을 검색함

```sql
SELECT 이름 FROM 학생
    WHERE 시험점수 >= 60 OR 과제 = 1;
```

4. 과제가 0 또는 1인 학생의 이름, 시험점수, 과제를 검색함

```sql
SELECT 이름, 시험점수, 과제 FROM 학생
    WHERE 과제 IN(0, 1);
```

5. 학점이 NULL인 학생의 이름과 시험점수를 검색함

```sql
SELECT 이름, 시험점수 FROM 학생
    WHERE 학점 IS NULL;
```

6. 이름이 '김'으로 시작되는 학생의 이름과 학점을 검색함

```sql
SELECT 이름, 학점 FROM 학생
    WHERE 이름 LIKE '김%';
```

7. 시험점수가 60점에서 90점 사이인 학생의 이름을 검색함

```sql
SELECT 이름 FROM 학생
    WHERE 시험점수 BETWEEN 60 AND 90;
```

<br>

### 4) SELECT의 부속, 복수 질의 명령
<테이블명 : 학생>

| &nbsp;&nbsp;학번&nbsp;&nbsp; | &nbsp;&nbsp;이름&nbsp;&nbsp; | &nbsp;&nbsp;학과코드&nbsp;&nbsp; |
|:-:|:-:|:-:|
| &nbsp;&nbsp;H01&nbsp;&nbsp; | &nbsp;&nbsp;최성호&nbsp;&nbsp; | &nbsp;&nbsp;A01&nbsp;&nbsp; |
| &nbsp;&nbsp;H02&nbsp;&nbsp; | &nbsp;&nbsp;김희정&nbsp;&nbsp; | &nbsp;&nbsp;A02&nbsp;&nbsp; |
| &nbsp;&nbsp;H03&nbsp;&nbsp; | &nbsp;&nbsp;이재춘&nbsp;&nbsp; | &nbsp;&nbsp;A01&nbsp;&nbsp; |
| &nbsp;&nbsp;H04&nbsp;&nbsp; | &nbsp;&nbsp;임승현&nbsp;&nbsp; | &nbsp;&nbsp;E01&nbsp;&nbsp; |
| &nbsp;&nbsp;H05&nbsp;&nbsp; | &nbsp;&nbsp;김현태&nbsp;&nbsp; | &nbsp;&nbsp;C07&nbsp;&nbsp; |
| &nbsp;&nbsp;H06&nbsp;&nbsp; | &nbsp;&nbsp;김유신&nbsp;&nbsp; | &nbsp;&nbsp;C07&nbsp;&nbsp; |
| &nbsp;&nbsp;H07&nbsp;&nbsp; | &nbsp;&nbsp;정용조&nbsp;&nbsp; | &nbsp;&nbsp;A01&nbsp;&nbsp; |
| &nbsp;&nbsp;H08&nbsp;&nbsp; | &nbsp;&nbsp;신사임&nbsp;&nbsp; | &nbsp;&nbsp;D03&nbsp;&nbsp; |
| &nbsp;&nbsp;H09&nbsp;&nbsp; | &nbsp;&nbsp;김유신&nbsp;&nbsp; | &nbsp;&nbsp;E01&nbsp;&nbsp; |
| &nbsp;&nbsp;H10&nbsp;&nbsp; | &nbsp;&nbsp;윤정연&nbsp;&nbsp; | &nbsp;&nbsp;E01&nbsp;&nbsp; |

<br>

<테이블명 : 학과>

| &nbsp;&nbsp;학과코드&nbsp;&nbsp; | &nbsp;&nbsp;학과명&nbsp;&nbsp; |
|:-:|:-:|
| &nbsp;&nbsp;A01&nbsp;&nbsp; | &nbsp;&nbsp;수학과&nbsp;&nbsp; |
| &nbsp;&nbsp;A02&nbsp;&nbsp; | &nbsp;&nbsp;물리학과&nbsp;&nbsp; |
| &nbsp;&nbsp;A03&nbsp;&nbsp; | &nbsp;&nbsp;화학과&nbsp;&nbsp; |
| &nbsp;&nbsp;A04&nbsp;&nbsp; | &nbsp;&nbsp;통계학과&nbsp;&nbsp; |
| &nbsp;&nbsp;C07&nbsp;&nbsp; | &nbsp;&nbsp;경제학과&nbsp;&nbsp; |
| &nbsp;&nbsp;C08&nbsp;&nbsp; | &nbsp;&nbsp;경영학과&nbsp;&nbsp; |
| &nbsp;&nbsp;D03&nbsp;&nbsp; | &nbsp;&nbsp;무역학과&nbsp;&nbsp; |
| &nbsp;&nbsp;E01&nbsp;&nbsp; | &nbsp;&nbsp;컴공과&nbsp;&nbsp; |
| &nbsp;&nbsp;E02&nbsp;&nbsp; | &nbsp;&nbsp;건축학과&nbsp;&nbsp; |
| &nbsp;&nbsp;E03&nbsp;&nbsp; | &nbsp;&nbsp;기계학과&nbsp;&nbsp; |

1. <학과> 테이블에서 학과명이 '수학과'를 찾아 <학생> 테이블에서 학과코드가 같은 학번과 이름을 검색함

```sql
SELECT 학번, 이름 FROM 학생 WHERE 학과코드 =
 (SELECT 학과코드 FROM 학과 WHERE 학과명 = '수학과');
```

2. <학생> 테이블의 학과코드와 <학과> 테이블의 학과코드가 'A01'로 같은 학생의 이름과 학과명을 검색함

```sql
SELECT 학생.이름, 학과.학과명 FROM 학생, 학과
    WHERE 학생.학과코드 = 'A01' AND 학과.학과코드 = 'A01';
```

<br>

### 5) SELECT의 그룹 지정 명령
<테이블명 : 성적테이블>

| &nbsp;&nbsp;학번&nbsp;&nbsp; | &nbsp;&nbsp;과목&nbsp;&nbsp; | &nbsp;&nbsp;학점&nbsp;&nbsp; | &nbsp;&nbsp;점수&nbsp;&nbsp; |
|:-:|:-:|:-:|:-:|
| &nbsp;&nbsp;100&nbsp;&nbsp; | &nbsp;&nbsp;자료 구조&nbsp;&nbsp; | &nbsp;&nbsp;A&nbsp;&nbsp; | &nbsp;&nbsp;90&nbsp;&nbsp; |
| &nbsp;&nbsp;100&nbsp;&nbsp; | &nbsp;&nbsp;운영체제&nbsp;&nbsp; | &nbsp;&nbsp;A&nbsp;&nbsp; | &nbsp;&nbsp;95&nbsp;&nbsp; |
| &nbsp;&nbsp;200&nbsp;&nbsp; | &nbsp;&nbsp;운영체제&nbsp;&nbsp; | &nbsp;&nbsp;B&nbsp;&nbsp; | &nbsp;&nbsp;85&nbsp;&nbsp; |
| &nbsp;&nbsp;300&nbsp;&nbsp; | &nbsp;&nbsp;프로그래밍&nbsp;&nbsp; | &nbsp;&nbsp;A&nbsp;&nbsp; | &nbsp;&nbsp;90&nbsp;&nbsp; |
| &nbsp;&nbsp;300&nbsp;&nbsp; | &nbsp;&nbsp;데이터베이스&nbsp;&nbsp; | &nbsp;&nbsp;C&nbsp;&nbsp; | &nbsp;&nbsp;75&nbsp;&nbsp; |
| &nbsp;&nbsp;300&nbsp;&nbsp;| &nbsp;&nbsp;자료 구조&nbsp;&nbsp; | &nbsp;&nbsp;A&nbsp;&nbsp; | &nbsp;&nbsp;95&nbsp;&nbsp; |
| &nbsp;&nbsp;400&nbsp;&nbsp; | &nbsp;&nbsp;소프트웨어&nbsp;&nbsp; | &nbsp;&nbsp;B&nbsp;&nbsp; | &nbsp;&nbsp;80&nbsp;&nbsp;|
| &nbsp;&nbsp;500&nbsp;&nbsp; | &nbsp;&nbsp;자료 구조&nbsp;&nbsp; | &nbsp;&nbsp;C&nbsp;&nbsp; | &nbsp;&nbsp;75&nbsp;&nbsp; |
| &nbsp;&nbsp;500&nbsp;&nbsp; | &nbsp;&nbsp;운영체제&nbsp;&nbsp; | &nbsp;&nbsp;D&nbsp;&nbsp; | &nbsp;&nbsp;65&nbsp;&nbsp; |
| &nbsp;&nbsp;500&nbsp;&nbsp; | &nbsp;&nbsp;프로그래밍&nbsp;&nbsp; | &nbsp;&nbsp;F&nbsp;&nbsp; | &nbsp;&nbsp;54&nbsp;&nbsp; |

1. <성적테이블>의 모든 속성을 대상으로 학번에 따라 그룹지어서 학번별로 튜플의 개수를 출력함

```sql
SELECT 학번, COUNT(*) FROM 성적테이블
    GROUP BY 학번;
```

2. <성적테이블>에서 학번에 따라 그룹지어 학번별로 튜플의 개수가 3 이상인 학생의 학번과 튜플 개수를 출력함

```sql
SELECT 학번, COUNT(*) FROM 성적테이블
    GROUP BY 학번 HAVING GOUNT(*) >= 3;
```

3. <성적테이블>에서 학번 속성을 대상으로 학번에 따라 그룹지어서 학번 별로 튜플의 개수를 출력하되 튜플 개수의 속성명을 학생수로 지정함(단, 학번별로 튜플의 개수가 3이상)

```sql
SELECT 학번, COUNT(학번) AS 학생수 FROM 성적테이블
    GROUP BY 학번 HAVING COUNT(학번) >= 3;
```

4. <성적테이블>에서 학번 속성을 대상으로 학번에 따라 그룹지어서 학번별로 튜플의 개수와 점수의 평균점수를 출력하되 튜플 개수의 속성명을 학생수, 평균점수의 속성명을 평균점수로 지정함(단, 학번별로 튜플의 개수가 3 이상)

```sql
SELECT 학번, COUNT(*) AS 학생수,
    AVG(점수) AS 평균점수 FROM 성적테이블
    GROUP BY 학번 HAVING COUNT(학번) >= 3;
```

<br>

### 6) SELECT의 테이블 생성 명령
1. SELECT의 테이블 생성 명령
    - 새롭게 생성하고자 하는 테이블이 기존에 사용하고 있는 테이블과 동일한 구조를 갖고 있다면 다음과 같이 기존에 존재하는 테이블 정보를 이용하여 새로운 테이블을 만들 수 있음
    - SQL문의 SELECT 부분을 통해 기존 테이블의 속성을 조회하여 신규 테이블의 속성으로 정의하여 생성하는 방식

```sql
CREATE TABLE 신규_테이블 AS SELECT * FROM 기존_테이블;
```

2. SELECT의 테이블 생성 명령의 특징
    - 생성된 테이블은 기존 테이블의 컬럼 및 데이터 유형과 길이 등을 그대로 적용함
    - NOT NULL의 정의는 그대로 적용함
    - 제약조건은 적용되지 않음
    - ALTER TABLE을 사용하여 제약조건을 추가해야 함
    - 동일한 컬럼들로 생성된 경우 '*'을 사용함
    - 필요한 컬럼만을 지정하여 테이블을 생성할 수 있음