# 데이터베이스 키

### 데이터베이스 키

* 슈퍼키: 유일성을 만족하는 키
* 복합키: 2개 이상의 속성을 사용한 키
* 후보키: 유일성과 최소성을 만족하는 키. 기본키가 될 수 있음.
* **기본키**: 후보키중 선택된 키. null값이 들어갈수없으며 동일한 값이 들어갈 수 없다.
* 대체키: 후보키중 선택되지 않은 키
* **외래키**: 다른 테이블의 기본키를 참조하는 키

### 무결성 제약 조건

데이터베이스의 정확성, 유효성을 의미

1. 도메인 제약 조건\
   각 속성들이 원자값(중복x)이여야함.
2. 키 제약 조건\
   키 속성이 중복된 값이면 안됨.
3. 엔티티 무결성 제약 조건\
   기본키 값이 널값을 가질수없음.
4. 참조 무결성 제약 조건\
   두 릴레이션에 연관된 튜플들 관계는 일관성을 유지해야함.\
   외래키 값은 참조테이블의 기본키 값이여야 한다.

#### 무결성 제약 조건 유지

dbms는 삽입, 삭제, 수정 연산에서 데이터베이스의 무결성 제약 조건을 만족하도록 함.\
부모테이블을 참조하는 자식테이블이 존재한다면

자식 삽입 -> 참조 무결성 검증 O\
자식 삭제 -> 참조 무결성 검증 X\
부모 삭제 -> 참조 무결성 검증 O\
부모 삽입 -> 참조 무결성 검증 X

#### 참조 무결성 옵션

restricted(제한): 참조 무결성에 위배되는 연산은 거절\
cascade(연쇄): 참조되는 튜플도 같이 연산함. 참조되는 튜플이 삭제되면 참조하는 튜플도 삭제.\
nullify: 참조되는 튜플이 삭제되면 참조하는 튜플에는 null값 삽입.\
default: null값 대신 default값 삽입.

### 테이블 제약 조건

테이블에 부적절한 값이 저장되는것을 방지하고자 정해놓은 규칙

1.  **not null** - null 값이 들어가면 안됨

    ```sql
    creat table mytable1( varchar(255) name NOT NULL, varchar(255) age);              
    ```
2.  **unique** - 중복된 값이 들어가면 안됨

    ```sql
    alter table mytable1 add constraint mytable1_unique UNIQUE(age);           
    ```
3.  check - 컬럼의 값을 특정 범위로 제한함

    ```sql
    alter table mytable1 add constraint mytable1_check CHECK(age > 0 and age < 100);           
    ```
4.  default - 컬럼에 값이 입력되지 않으면 기본값으로 지정함

    ```sql
    alter table mytable1 alter name SET DEFAULT "홍길동";               
    ```
5.  primary key - 기본키 지정. not null + unique\
    기본키를 지정하면 인덱스가 생성됨

    ```sql
    create table mytable1( varchar(255) name PRIMARY KEY);      
    alter table mytable1 add constraint mytable1_primary_key PRIMARY KEY(name);     
    ```

    테이블에 유티크한 값이 없다면 필드를 묶어서 기본키 지정 가능

    ```sql
    constraint mytable2_primary_key PRIMARY KEY(field2, field2);                
    ```



