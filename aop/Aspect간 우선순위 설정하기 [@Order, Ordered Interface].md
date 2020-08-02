## Aspect간 우선순위 설정하기 [@Order, Ordered Interface]

AOP 기법을 적용하기 위해서 스프링에서는 @Aspect를 작성하여 여러 어드바이스를 구현합니다.
이럴 경우 같은 종류의 어드바이스가 설정된 @Aspect 클래스가 존재할 경우
어떤 애스펙트 클래스의 어드바이스를 실행해야 할지 정할 수가 없다.
그래서 어떤 애스펙트 클래스의 어드바이스를 실행할지를 정하기 위하여 스프링이 제공하는
인터페이스와 어노테이션이 존재한다.

### 1. Ordered 인터페이스 사용하기

스프링에서는 애스펙트 클래스 간의 우선순위를 결정하기 위해서 Ordered 인터페이스를 지원한다.
Ordered 인터페이스는 다음과 같은 패키지에 존재한다.

```java
org.springframework.core
```

Ordered 인터페이스를 사용하는 방법은 우선순위를 적용할 애스펙트 클래스가
Ordered 인터페이스를 구현하도록 만들면 된다.

Ordered 인터페이스는 getOrder() 메소드를 구현하도록 하는데
이 메소드가 반환하는 int 타입의 리턴값이 작으면 작을 수록 우선순위가 높다.

```java
@Aspect
@Component
public class AspectTest1 implements Ordered {
    public int getOrder() {
        return 0;
    }
}
```

```java
@Aspect
@Component
public class AspectTest2 implements Ordered {
    public int getOrder() {
        return 1;
    }
}
```

위와 같이 AspectTest1 클래스와 AspectTest2 클래스가 존재할 경우
AspectTest1 클래스가 더 높은 우선순위를 갖는다.

### 2. @Order 사용하기

@Order은 모든 애스펙트 클래스가 구현하면서 생기는 복잡함을 해결하기 위해서
이를 어노테이션으로 지원한 것이다.
@Order의 value 값으로는 int 타입의 우선순위를 받는데
이 int 타입의 우선순위가 낮으면 낮을 수록 높은 우선순위에 배정된다.

```java
@Aspect
@Component
@Order(0)
public class AspectTest1 { ... }
```

```java
@Aspect
@Component
@Order(1)
public class AspectTest2 { ... }
```

따라서 이는 AspectTest1이 더 높은 우선순위를 갖는다.