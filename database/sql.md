# SQL (Structured Query Language)

![sql](./database_img/sql_img/sql.png)



- 관계형 데이터베이스의 데이터를 관리할 목적으로 설계된 특수한 프로그래밍 언어
- 1970년대 초 IBM에서 개발 (SEQUEL)



## SQL 문법의 큰 구분

#### (1) 데이터 정의 언어 (DDL : Data Defenition Language)

- 데이터 저장을 위한 테이블 구조와 형식을 정의하기 위한 문법

#### (2) 데이터 조작 언어 (DML : Data Manipulation Language)

- 데이터를 읽고, 쓰고, 수정하고, 삭제하기 위한 문법 - DBA가 아닌 개발자도 다루어야 함

#### (3) 데이터 제어 언어 (DCL : Data Control Language)

- 데이터의 실행을 제어하기 위한 문법



## DML 문법 기본



#### INSERT

```sql
INSERT INTO 테이블명(칼럼1, 칼럼2, 칼럼3..)
VALUES (값1, 값2, 값3..)
	   (값1, 값2, 값3..)
```



#### SELECT

```sql
SELECT 읽어들일 칼럼1, 칼럼2, 칼럼3..
FROM 테이블명
[WHERE 검색할 데이터의 조건]
[ORDER BY 데이터 정렬 조건]
```



#### UPDATE

```sql
UPDATE 테이블명
SET
	수정할 칼럼1 = 수정할 값1
	수정할 칼럼2 = 수정할 값2
	수정할 칼럼3 = 수정할 값3
	...
	[WHERE 수정할 데이터의 조건] 
```



#### DELETE

```sql
DELETE [삭제할 칼럼명] FROM 테이블명 [WHERE 삭제할 데이터의 조건]
```



#### JOIN

```sql
SELECT e.emp_cd, e.emp_name, e.emp_class, e.join_date, e.depart_cd
FROM employee e <- 기준
JOIN department d ON d.depart_cd = e.depart_cd
...
```

*INNER JOIN 방법임*



- 특정 칼럼값만 삭제해야 하는 (흔치 않은) 경우에는 UPDATE 구문을 사용해 처리하는 것이 일반적
- 대부분의 DELETE 칼럼에서는 칼럼명은 생략되어 해당 데이터(로우) 전체가 삭제됨

- MySQL이나 SQLServer의 경우 명시적 커밋 명령이 아니더라도 반영될 수 있기 때문에 조심
- SELECT, UPDATE, DELETE 모두에서 WHERE 조건문을 필수적으로 사용하자



## DBSM에서의 SQL 구문 실행 과정

#### (1) Parsing

- SQL 구문의 문법과 의미가 정확한지 검사
- 이상이 없다면 내부 캐시에 동일한 SQL 문장이 저장되어 있는지 확인, 저장되어 있을 경우 기존 컴파일 정보를 이용하여 2,3단계를 건너뛰고 4단계를 바로 실행한다

#### (2) Optimizing

- 데이터 분포 통계나 테이블의 관계 정보를 바탕으로 주어진 SQL 문에 최적화된 실행 계획(Execution Plan) 수립
- 데이터베이스 내부 옵티마이저 활용
- 리소스를 많이 잡아먹는 단계

#### (3) Compilation

- 옵티마이저에 의해 수립된 실행 계획을 바탕으로 단계별 실행 계획을 생성하여 바이너리 코드로 저장
- 바이너리 코드는 트리 구조로 생성된 후 재사용을 위해 내부 캐시에 저장됨

#### (4) Execution

- 생성된 실행 계획을 DBMS 엔진에서 수행하여 결과를 사용자에게 반환



*기존 실행 계획이 있을 경우 2,3 단계가 생략될 수 있지만, 구문을 분석하고 실행 계획을 (해쉬)검색하는 파싱 단계는 SQL의 수행 횟수만큼 반복될 수 밖에 없다는 한계가 존재함.*



## Prepared Statement(선처리된 구문)

```sql
INSERT INTO department (depart_title, depart_addr) VALUES ( ? , ? )
```

- 위치홀더(Placeholder)를 통해 SQL 쿼리를 단순화한 형태

- SQL 구문과 인자값을 분리하여 전달하여 DBMS의 SQL 수행 성능을 향상시킴

- 파레토의 법칙(Pareto's rule)에 근거하여 동일 SQL문 반복 수행을 효율적으로 만들기 위하여 사용

- > 상위 20%가 전체 생산의 80%를 해낸다
  >
  > 자주 사용되는 20%의 SQL이 전체 내용 중 80%를 차지한다. 특정 SQL문의 반복 수행 빈도가 높다



탬플릿 형식으로 작성된 SQL을 먼저 전송하여 실행 계획을 컴파일하고 뒤이어 매개변수를 전송하는 방식

컴파일된 실행 결과를 가지고 있다가 인자값만 넣고 실행함으로써 구문 분석과 실행 계획의 검색 컴파일 모두 생략. 단순 실행만. 

미리 컴파일된 실행 계획에 인자값을 순서대로 전송하여 바로바로 질의를 수행함

오버헤드는 맨 처음 한 번만 발생. 원하는 만큼 다른 값으로 구문 실행 가능

<br>

>  *인자값을 ?로 대체하는 것이 어떻게 실행 계획 재사용을 가능케 하는가?*

- DBMS는 일반적으로 실행 계획의 빠른 검색을 위해 해싱 알고리즘을 사용함

- DBMS는 주어진 SQL문을 통째로 해시 연산한 결과값으로 저장 위치를 결정하고 실행 계획을 저장함

- 따라서 SQL 문의 인자값 하나만 바뀌어도 저장 위치는 180도 달라질 수 있음. 실행 계획 재사용 불가능.

- Prepared Statement는 구체적인 인자값을 제거하고 이를 위치홀더로 대체했기 때문에, 인자값이 변해도 두 구문의 해쉬값이 동일하며, 실행 계획을 재사용하는 것도 아무 문제가 없음. 

  > 아예 동일한 SQL문 뿐만 아니라, 인자값이 다르지만 구조가 동일한 SQL문에 대해서도 효율적인 재사용이 가능한 것. 
  >
  > (1) 처음 SQL 구문 전달시에만 1~3단계를 한 번 거치고 그 다음부터는 인자값 순서대로 4단계 실행만 해주면 된다 (statement 방법일 경우 인자값까지 같은 쿼리라도 매번 1, 4단계를 반복해줘야 함. 파싱에 걸리는 시간이 절약됨)
  >
  > (2) 인자값이 변하더라도 아무 상관 없음 (statement 방법일 경우 인자값이 달라지면 아예 처음부터 1~4단계를 새로 진행해야 함)



#### Prepared Statement의 이점

(1) SQL을 여러번 실행하더라도 SQL을 컴파일하고 최적화하는 오버헤드는 단 한 번만 발생

(2) 매개변수를 변조하여 SQL문을 변형하는 <u>SQL Injection</u> 위험으로부터 안전

(3) SQL에는 위치홀더만 포함되기 때문에 인자값에 대한 쿼우팅 처리를 따로 할 필요가 없음



#### Prepared Statement의 단점

(1) 최적화 단계에서 인자값의 특성이 반영되지 않으므로, 실제 데이터 분포에 대한 히스토그램 데이터를 활용할 수 없어 비효율적인 실행 계획이 생성될 가능성이 있음

(2) 단발성 쿼리의 경우 DBMS에 대한 추가 왕복 작업으로 불필요한 리소스가 소비될 수 있음 (완성된 SQL문을 한 번만 전송하는 대신, SQL문을 전송하고 인자값을 추가적으로 전송하는 두 번의 처리)

<br>

## Reference

꼼꼼한 재은씨의 Swfit 실전편