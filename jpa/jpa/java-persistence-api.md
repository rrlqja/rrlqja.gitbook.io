# Java Persistence Api

### JPA - Java Persistence Api

java ORM 표준

Java Application과 JDBC 중간에서 동작한다.

> _ORM_ (Object Relational Mapping)
>
> 객체와 관계형 데이터베이스를 매핑하는 기술
>
> 객체는 객체대로, DB는 DB대로 설계하고 ORM 프레임워크가 중간에서 매핑



#### JPA 사용 이유

* sql 중심적 개발에서 객체 중심 개발 가능
* 객체와 데이터베이스의 패러다임 불일치를 해결

#### 패러다임의 불일치

객체지향 프로그래밍의 경우 추상화, 캡슐화, 상속, 다형성 등 다양한 객체 제어 방법이 존재하지만 관계형 데이터베이스는 존재하지 않는다.&#x20;

jpa는 이러한 패러다임의 불일치를 해결할 수 있다.

#### jpa 상속

객체는 상속이 존재하지만 관계형 데이터베이스는 상속 개념이 존재하지 않는다. 상속관계의 객체를 테이블에 저장, 조회하려면 관련된 쿼리를 여러개 생성해야한다. jpa는 이러한 문제를 해결할 수 있다.

#### Jpa 연관관계, 객체 그래프 탐색

객체는 참조를 활용, 테이블은 외래키를 활용하여 관련 객체, 테이블을 조회한다. jpa는 \<b>entity의 필드에 다른 entity의 참조를 저장\</b>하여 연관관계를 매핑한다.

#### Jpa 비교

동일한 트랜잭션에서 조회한 entity는 같음(==)을 보장한다.

#### Jpa 성능 최적화

1. 1차 캐시와 동일성 보장&#x20;
   1. 같은 트랜잭션안에서 같은 entity를 반환
   2. DB  Isolation Level이 Read Commit인 경우도 애플리케이션에서 Repeatable Read 보장
2. 트랜잭션을 지원하는 쓰기 지연
   1. 트랜잭션을 커밋할 때까지 insert 쿼리 모아 놓음
   2. Jdbc Bath Sql을 사용하여 한번에 쿼리를 전송
3. 지연 로딩과 즉시 로딩
   1. 지연 로딩: 객체가 실제 사용될 때 로딩
   2. 즉시 로딩: join sql로 연관된 엔티티를 한번에 로딩



***

### 영속성 컨텍스트(Persistence Context)

* 엔티티(entity)를 영구 저장하는 환경
* 눈에 보이지 않는 논리적 개념
* 엔티티 매니터(entity manager)를 통해 영속성 컨텍스트에 접근

#### 엔티티의 생명주기

* 비영속(new/transient): 영속성 컨텍스트와 전혀 관계가 없는 상태, 새로운 객체를 생성만 한 단계
* 영속(managed): 영속성 컨텍스트에 관리되는 상태. 새로운 객체를 엔티티 매니저를 통해 저장, 조회 등
*   준영속(detached): 영속성 컨텍스트에 저장되었다가 분리된 상태

    ```java
    entitymanager.detach(entity);   
    ```
*   삭제(removed): 영속성 컨텍스트에 삭제된 상태

    ```java
    entitymanger.remove(entity);    
    ```

#### 영속성 컨텍스트의 이점

* 1차 캐시
  * 엔티티 저장시 1차 캐시에 먼저 저장됨
  * 엔티티 조회시 1차 캐시에서 먼저 조회 후 캐시에 없을 경우에 쿼리를 생성함
*   동일성 보장

    * 영속 엔티티의 동일성을 보장함

    ```java
    member1 = entitymanger.find(member.class, member1ID);
    member2 = entitymanger.find(member.class, member1ID);       
    member1 == member2; // true
    ```
* 트랜잭션을 지원하는 쓰기 지연
  * entityManager.persist()를 호출해도 sql이 데이터베이스에 전송되지 않고 쓰기 지연 sql 저장소에  sql을저장함
  *   트랜잭션 커밋 시점에 쓰기 지연 sql 저장소의 쿼리를 데이터베이스에 전송     &#x20;

      ```java
      entitymanager.persist(entityA);
      entitymanager.persist(entityB);
      //sql을 전송하지 않음

      //커밋하는 순간에 sql전송
      transaction.commit()//커밋    
      ```
* 변경 감지(Dirty Checking)
  *   엔티티를 조회 후 필드값을 변경한 후에 따로 update 쿼리를 생성하지 않아도 변경 감지를 통해 update 쿼리를 생성함

      ```java
      member = entitymanger.find(member.class, memberID);
      member.setName("홍길동");

      //entitymanager.update(member) 이러한 코드를 작성하지 않아도 됨.

      transaction.commit()// 자동으로 update 쿼리를 보냄.     
      ```

      > 트랜잭션이 커밋되면 플러시가 발생함. 플러시가 발생되면 1차 캐시 내부의 스냅샷과 엔티티를 비교 후 update sql을 생성후 데이터베이스에 보냄.
* 지연 로딩(Lazy Loading)

#### 플러시(Flush)

* 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영
* 플러시 발생시
  * 변경 감지
  * 수정된 엔티티를 쓰기지연 sql 저장소에 저장
  * 쓰기 지연 sql 저장소의 쿼리를 데이터베이스에 전송
* 플러시 발생
  * entityManager.flush() - 플러시 직접 호출
  * 트랜잭션 커밋 - 플러시 자동 호출
  *   jpql 쿼리 실행 - 플러시 자동 호출

      ```java
      entitymanager.persist(memberA)
      entitymanager.persist(memberB)
      entitymanager.persist(memberC)

      //중간에 jpql 실행
      entitymanager.createQuery(query)//멤버 엔티티가 저장이 안된 상태에서 쿼리를 보내면 원치않는 결과가 나올 수 있음.
                                      //이러한 결과를 방지하고자 jpql 실행시 플러시가 자동으로 호출됨.
      ```

#### 준영속 상태

* 영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 상태
* 영속성 컨텍스트가 제공하는 기능을 사용하지 못함
* 준영속 상태를 만드는 방법
  1. entitymanger.detach(entity) - 특정 엔티티만 준영속 상태로 만듬
  2. entitymanger.clear() - 영속성 컨텍스트를 완전 초기화
  3. entitymanger.close() - 영속성 컨텍스트를 종료



***

### 엔티티 매핑

* 객체와 테이블을 매핑: `@Entity`&#x20;
  * `@Entity`: `@Entity`가 붙은 클래스는 jpa가 관리하는 엔티티라고 한다.
  * jpa를 사용해서 테이블과 매핑할 클래스는 `@Entity`필수&#x20;
  * 기본 생성자 필수(public, protected)
  * final, enum, interface, inner 클래스 X
  *   저장할 필드에 final 사용 X

      ```java
      @Entity //jpa entity
      public class ExEntity{
          @Id
          private Long Id;

          public ExEntity(){
          }
      }
      ```
*   필드와 컬럼 매핑

    ```java
    @Entity
    public class ExEntity{
        @Id
        private Long Id;

        @Column(name="username")// 컬럼 매핑
        private String userName;

        @Enumerated(EnumType.String)// enum 타입 매핑
        private EnumClass enumClass;

        @Temporal(TemporalType.TIMESTAMP)// 날짜 타입 매핑
        private Date createDate;

        @Lob// BLOB, CLOB 매핑
        private String description;

        @Transient// 필드를 컬럼에 매핑하지 않을때
        private int temp;
    }
    ```
* 기본키 매핑
  * `@Id`&#x20;
  * `@GeneratedValue`: 기본키 생성을 데이터베이스에 위임
    * Identity: 데이터베이스에 위임. mysql
    * Sequence: 데이터베이스 시퀀스 오브젝트 사용. oracle
      * `@SequenceGenerator` 필요
    * Table: 키 생성용 테이블 사용. 모든 DB
      * `@TableGenerator` 필요
    *   Auto: 데이터베이스 방언에 따라 자동 지정

        ```java
        @Entity
        public class ExEntity{
            @Id
            @GeneratedValue(strategy = GenerationType.Auto)
            private Long id;
        }
        ```
* 연관관계 매핑: `@ManyToOne`, `@JoinColumn`



***

### 연관관계 매핑 기초



























