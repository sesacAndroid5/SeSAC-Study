# SQL 개요 및 정리

## 1. SQL이란?
- **SQL (Structured Query Language)**
    - 관계형 데이터베이스 관리 시스템(RDBMS) 표준 질의언어
    - **ANSI, ISO**에서 표준 언어로 채택
    - **관계대수와 관계해석** 기반
    - **데이터 정의(DDL), 조작(DML), 제어(DCL) 기능 제공**
    - 비절차적, 대화식 언어
    - 다양한 프로그래밍 언어와 연동 가능

---

## 2. SQL 제공 데이터 타입

| 구분   | 타입(예시)            | 설명                 |
|------|---------------------|--------------------|
| 비트형 | BIT(n), VARBIT(n)    | Bit열, 고정/가변길이   |
| 문자형 | CHAR(n), VARCHAR(n)  | 문자열, 고정/가변길이  |
| 수치형 | SMALLINT, INT, FLOAT | 정수, 실수 자료형     |
| 10진형 | DECIMAL(i, j)        | 소수점 지원           |
| 기타   | DATE, TIME, LOGICAL, GRAPHIC | 날짜, 시각, 논리값 등     |

---

## 3. SQL 명령문의 분류

| 분류     | 명령어                      | 설명                        |
|--------|--------------------------|---------------------------|
| DDL    | CREATE, ALTER, DROP      | 데이터 구조 정의, 변경, 삭제   |
| DML    | SELECT, INSERT, UPDATE, DELETE | 데이터 조회, 입력, 수정, 삭제   |
| DCL    | GRANT, REVOKE, COMMIT, ROLLBACK | 권한, 트랜잭션 관리          |

- **주석**
    - 한 줄: `--`
    - 여러 줄: `/* ... */`

---

## 관계형 데이터베이스 용어 정리

| 용어          | 설명                                | 테이블에서 대응되는 개념             |
|---------------|-----------------------------------|-----------------------------------|
| 튜플 (Tuple)   | 한 행(row), 하나의 데이터 단위      | 테이블의 한 행(row) / 레코드(record) |
| 릴레이션 (Relation) | 2차원 표, 즉 테이블 자체           | 테이블(Table) 전체                 |
| 속성 (Attribute) | 튜플을 구성하는 하나의 필드(컬럼)   | 테이블의 열(column)                |
| 도메인 (Domain) | 속성이 가질 수 있는 값들의 집합      | 컬럼이 허용하는 값의 범위          |
| 카디널리티 (Cardinality) | 튜플(행)의 개수                   | 테이블의 행 수                    |
| 차수 (Degree)   | 테이블의 속성 개수                  | 테이블의 열 수                    |

<img width="690" height="227" alt="image" src="https://github.com/user-attachments/assets/030e8513-f01e-489f-a0a7-2bd7430df3b8" />




## 4. 주요 SQL 명령문 예시

## DDL
### (1) 테이블 생성 [CREATE]
**명령**: CREATE TABLE 테이블명 (속성명1 타입 [PRIMARY KEY], 속성명2 타입, …);

```
CREATE TABLE Student (
StudentID int NOT NULL PRIMARY KEY,
Name varchar(255),
Gender varchar(255),
Address varchar(255)
);
```


### (2) 뷰 생성 [CREATE]
**명령**:  CREATE VIEW 뷰이름 (속성명1, 속성명2) AS SELECT (속성1, 속성2,…) FROM 테이블명 [WHERE 조건식];

```
CREATE VIEW MyStudent(StudentID, Name, Gender)
AS SELECT StudentID, Name, Gender FROM Student WHERE Gender = '남';
```

### (3) 테이블 변경 [ALTER]
**명령**: ALTER TABLE 테이블명 {ADD , ALTER, DROP} 속성 타입;

```
ALTER TABLE Student ADD Score INT NOT NULL;
```


### (4) 테이블 / 뷰 / 인덱스 삭제 [DROP]
**명령**:  DROP {TABLE, VIEW, INDEX} 대상체 [CASCADE, RESTRICT];
    - CASCADE: 참조되는 모든 테이블/뷰 삭제
    - RESTRICT: 지정 대상체만 삭제 (기본값)

```
DROP TABLE Student;
DROP VIEW MyStudent;
```


## DML
### (5) 데이터 조회 (SELECT)
**명령**: SELECT [ALL, DISTINCT(중복제외)] 속성들 FROM 테이블명 [WHERE 조건식] […];

- 전체 조회
```
  SELECT * FROM 학생;
 ```


- 조건 검색  
```
  SELECT * FROM 학생 WHERE 성별 = '남';
```


- 중복 제거  
```
  SELECT DISTINCT 성별 FROM 학생;
```


- 정렬
```
  SELECT * FROM 학생 ORDER BY 학번 DESC;
```


### (6) 데이터 삽입 (INSERT)
**명령**: INSERT INTO 테이블명 ([속성1,속성2,…]) VALUES (속성1값[,속성2값2,…]);

```
INSERT INTO 학생(학번, 이름, 성별, 출생년도)
VALUES (2021006, '이하이', '여', 1996);
```


### (7) 데이터 수정 (UPDATE)
**명령**: UPDATE 테이블명 SET 속성=값[속성2=값2,…] [WHERE 조건식]

```
UPDATE 학생 SET 출생년도 = 1992 WHERE 학번 = 2021002;
```


### (8) 데이터 삭제 (DELETE)
**명령**: 명령: DELETE FROM 테이블명 [WHERE 조건식]
```
DELETE FROM 학생 WHERE 성별 = '남';
```


---

## 5. 권한 제어 (DCL)
- **권한 부여** [GRANT]
**명령**: GRANT 명령어 ON 테이블명 TO 사용자;

```
  GRANT SELECT ON STUDENT TO PUBLIC;
```


- **권한 회수** [REVOKE]
**명령**: REVOKE 명령어 ON 테이블명 FROM 사용자;

```
  REVOKE SELECT ON STUDENT FROM 사용자명;
```
