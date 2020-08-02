## 애스펙트 포인트컷 재사용하기 [@Pointcut]

@Aspect를 클래스에 작성하여 그 클래스를 애스펙트로 만들고,
그 안에 어드바이스를 작성해서 AOP를 구현했었다.
그리고 그 어드바이스에는 어노테이션의 요소로 포인트컷을 설정했었는데
같은 포인트컷이 여러 어드바이스에서 사용된다면 그는 재사용을 할 필요가 있다.
그래서 사용되는 어노테이션이 @Pointcut이다.

### 1. @Pointcut 사용하기

@Pointcut을 작성한 메소드가 있다면 그 메소드의 이름을 가지고 포인트컷을 사용할 수 있다.
이렇게 말하면 이해하기 어려울 수 있으니 예제로 살펴보자.

```java
@Aspect
@Component
public class AspectTest {
    @Pointcut("execution(* *.*(..))")
    private void all() {}
    
    @Before("all()")
    public void before(JoinPoint joinPoint) { ... }
    
    @AfterThrowing(
    	pointcut = "all()",
        throwing = "error"
    )
    public void afterThrowing(JoinPoint joinPoint, Throwable error) { ... }
}
```

이렇게 재사용할 포인트컷을 @Pointcut에 작성하고
작성한 메소드의 이름으로 포인트컷을 재사용할 수 있다.
이렇게 클래스 내부에서만 사용할 것이라면 사용할 메소드는 private으로 정의해놓는 것이 좋으며
바디는 없는 것이 깔금하다.

### 2. 다른 클래스에서 포인트컷 재사용하기

재사용해야할 포인트컷이 많아지고 그게 클래스 단위로 많아진다면
재사용할 포인트컷만 모아놓은 클래스가 생기게 될 것이다.
이럴 때는 private이 아니라 public으로 해야 포인트컷을 다른 클래스에서도 사용할 수 있으며
접근할 때 클래스.메소드명() 으로 접근해야 한다.
만약 다른 패키지에 존재한다면 모든 패키지명까지 적어주어야 한다.

```java
package abc.def.ghi;

@Aspect
@Component
public class PointcutMoa {
    @Pointcut("execution(* *.*(..))")
    public void all();
}
```

```java
package jkl.mno.pqr;

@Aspect
@Component
public class Test {
    @Before("abc.def.ghi.PointcutMoa.all()")
    public void before(JoinPoint joinPoint) { ... }
}
```

