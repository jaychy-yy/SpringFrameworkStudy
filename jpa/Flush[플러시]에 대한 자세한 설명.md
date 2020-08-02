## Flush[플러시]에 대한 자세한 설명

### 1. 개요

Flush(플러시)는 영속성 컨텍스트에서 변경된 내용은 데이터베이스에 반영하는 역할을 한다.
이는 영속성 컨텍스트를 가지고 있는 EntityManager(엔티티 매니저)가 가지고 있는
flush() 메소드를 말하기도 한다.

플러시를 사용하는 방법은 세 가지가 존재한다.

### 2. EntityTransaction.commit() 메소드를 호출한다.

EntityManager를 이용하면 다음과 같이 EntityTransaction을 얻을 수 있다.

```java
EntityManagerFactory entityManagerFactory =
    Persistence.createEntityManagerFactory("...");
EntityManager entityManager = entityManagerFactory.createEntityManager();
EntityTransaction entityTransaction = entityManager.getTransaction();
```

이렇게 얻은 EntityTransaction은 트랜잭션을 시작 및 끝낼 수 있다.
JPA를 이용한 데이터베이스 접근은 언제나 트랜잭션 안에서 해야 하므로
EntityTransaction은 계속 생성할 것이니 기억하는 게 좋다.

EntityTransaction은 begin() 메소드를 이용해서 트랜잭션을 시작할 수 있고,
commit() 메소드를 이용하여 트랜잭션을 끝내고 커밋을 할 수 있다.

그런데 이 commit() 메소드를 실행할 때 바로 EntityManager의 flush() 메소드를 호출하게 된다.
이는 커밋은 했는데 플러시를 안 해서 데이터베이스에 반영이 되지 않는 경우를 막기 위해서이다.
만약 이렇게 자동으로 flush() 메소드를 호출하지 않았더라면
데이터베이스에 반영하기 위해서 두 가지 동작을 해야 하므로 더 효율적인 측면을 위해 구성된듯하다.

### 3. EntityManager.flush() 메소드를 호출한다.

계속 얘기하지만 flush() 메소드는 정적 메소드가 아니라 표기상 저런 형식으로 설정해놓은 것이다.
영속성 컨텍스트는 EntityManager가 관리하므로 당연히 EntityManager를 이용해서
flush() 메소드를 호출할 수 있을 것이다.

그런데 위에서 EntityTransaction으로 인한 commit() 메소드가 호출될 때 flush() 메소드도 호출되므로,
임의적으로 이를 호출해야 하는 경우는 거의 없을 것이다.

하지만 테스트 코드를 짜면서 일부분 사용되는 부분은 있을 수 있다고 한다.

#### 4. JPQL을 사용한다.

JPA를 사용하면서 식별자(PRIMARY KEY)를 이용한 find() 메소드는 쉽게 찾을 수 있으므로
JPQL을 사용하지 않고도 찾는 것이 가능하다.
하지만 다른 값을 통해 찾을 경우 여러 값이 나올 수 있고 복잡하기 때문에 JPQL을 사용해야 한다.

JPQL은 Java Persistence Query Language의 약자로 JPA를 사용하면서
데이터베이스에 접근하기 위해서 데이터베이스 관점에서가 아니라 객체지향적인 부분에서 볼 수 있도록
지원하는 Query Language이다.

이러한 JPQL을 사용할 때 왜 flush()가 호출될까?
이런 상황이 있을 수도 있다.
만약 엔티티를 영속성 컨텍스트에 삽입(INSERT)하는 과정이 벌어지는데
그 후에 JPQL을 이용해서 여러 엔티티를 찾고자 한다면 찾을 때의 대상에서
1차 캐시에 존재하는, 아직 커밋이 되지 않은, 데이터베이스에 반영되지 않은 엔티티는 빠지게 된다.
그러면 찾고자 하는 대상에 빠진 엔티티는 가져온 리스트에 존재하지 않으므로
논리적으로 맞지 않는 느낌을 받을 수 있다.
그렇기 때문에 JPQL을 사용하게 되면 자동으로 flush() 메소드가 호출된다.

그리고 find() 메소드를 이용한 검색(SELECT)은 flush() 메소드가 호출되지 않는데
그 이유는 식별자를 이용한 검색은 굳이 데이터베이스에 존재하지 않아도
식별자만 비교하면 되므로 JPQL에 비해서 쉽게 계산이 가능하므로 검색대상에 포함된다.