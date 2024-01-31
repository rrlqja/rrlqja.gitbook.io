# @OneToOne 관계의 Lazy 로딩

### 문제 발생

프로젝트 진행중 문제가 발생했다.

```sh
select 
    s1_0.id,
    ...
from 
    sns s1_0
where 
    s1_0.user_id=?

select 
    s1_0.id,
    ...
from 
    sns s1_0
where 
    s1_0.user_id=?

select 
    s1_0.id,
    ...
from 
    sns s1_0
where 
    s1_0.user_id=?
```

동일한 쿼리가 여러번 생성되는 `N+1` 문제가 발생한 것이다.

`@xxToOne` 관계는 fetch join을 통해 즉시 로딩으로 조회하고 `@xxToMany` 관계는 batch 를 통해 최적화 하였다. 그런데도 `N+1` 문제가 발생했다.

엔티티 관계를 보면

```java
@Entity
public class Board {
    @JoinColumn(name = "writer_id")
    @ManyToOne(fetch = FetchType.LAZY)
    private User Writer;

    // others
}

@Entity
public class User {
    @OneToMany(mappedBy = "writer", fetch = FetchType.LAZY)
    private List<Board> boardList = new ArrayList<>();    

    @OneToOne(mappedBy = "user", fetch = FetchType.LAZY)
    private Sns sns;

    // others
}

@Entity
public class Sns {
    @JoinColumn(name = "user_id")
    @OneToOne(fetch = FetchType.LAZY)
    private User user;    

    //others
}
```

Board ->`@ManyToOne`-> User ->@OneToOne-> Sns 로 관계가 맺어져있다.

10개의 Board 리스트를 가져오고  fetch join을 통해 User를 함께 조회한다.

```java
@Repository
public interface BoardJpaRepository extends JpaRepository<Board, Long> {
    @EntityGraph(attributePaths = {"writer"})
    Page<Board> findAll(Pageable pageable);
}
```

예상하기로는 당연히 Board 리스트와 writer에 해당하는 User 엔티티를 fetch join으로 가져오고 사용하지 않는 Sns는 가져오지 않을거로 예상했다.

하지만 위에서 보듯이 Sns를 가져오는 쿼리가 Board의 작성자 수 만큼 생성된 것이다.

게다가 설정된 batch size 만큼 조회하는 것도 아닌 각각의 Board의 작성자별로 각각 조회한다.



### 원인

원인은 jpa 프록시에 있었다.

현재는 Sns와 User엔티티의 연관관계에서 Sns가 연관관계 주인으로 지정되어있다.

```java
@Entity
public class User { 
    @OneToOne(mappedBy = "user", fetch = FetchType.LAZY)
    private Sns sns;

    // others
}

@Entity
public class Sns {
    @JoinColumn(name = "user_id")
    @OneToOne(fetch = FetchType.LAZY)
    private User user;    

    //others
}
```

이러한 상태에서 jpa가 User를 조회할 때 문제가 발생한 것이다. 왜 문제가 발생한 것일까?

jpa가 User를 조회할 때 연관관계가 맺어져있는 Sns에 값을 할당해줘야 하는데 Sns 외래키를 확인하여 값이 존재 한다면 프록시 객체를 할당하고,   Sns 외래키가 존재하지 않는다면 null을 할당한다.

하지만 보이듯이 연관관계의 주인은 Sns에 지정되어있고, 즉 User테이블에 Sns의 외래키가 존재하는 것이 아닌 Sns테이블에 User의 외래키가 존재하는 상태이다.

User를 조회할때 User 테이블에는 Sns 외래키가 존재하지 않으니 Sns가 존재하는지 존재하지 않는지 jpa는 알 수 없다. 그러므로 해당 User의 Id를 외래키를 갖는 Sns 테이블을 각각 조회해서 Sns 엔티티가 존재하는지 조회한 것이다.



### 해결

jpa의 조회 상황을 두 가지로 요약해보면

1.  연관관계 주인이 Sns에 있을때

    User 테이블에는 Sns 외래키를 갖는 속성이 없다. 결국 User의 Id를 외래키를 갖는 Sns를 각각 검색한다. Sns 테이블 검색 결과를 User엔티티의 Sns 참조에 할당한다.\
    결국 User 검색 + Sns 검색 두 번의 검색이 이루어진다.
2. 연관관계 주인이 User에 있을때\
   User 테이블에 Sns 외래키가 존재한다. User 엔티티를 조회할 때 Sns 외래키를 확인하여 Sns 존재를 알 수 있어 프록시 혹은 null을 바로 할당할 수 있다.

이 두 가지로 요약할 수 있다.

결국해결 법은 단순히 연관관계의 주인은 바꾸어주면 된다.

User 엔티티가 연관관계의 주인으로 지정되면 User 테이블에 Sns 외래키가 할당되고, User를 조회할때 Sns 외래키를 바로 확인하여 프록시 혹은 null을 할당할 수 있다.

```java
@Entity
public class User { 
    @JoinColumn(name = "sns_id")
    @OneToOne(fetch = FetchType.LAZY)
    private Sns sns;

    // others
}

@Entity
public class Sns {
    @OneToOne(mappedBy = "sns", fetch = FetchType.LAZY) // 연관관계 주인 변
    private User user;    

    //others
}
```











