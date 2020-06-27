## 어노테이션으로 AOP 구현하기 [@EnableAspectJAutoProxy, @Aspect, @Before, @After, @AfterReturning, @AfterThrowing, @Around]

전에 자바로 AOP를 구현해보고 스프링의 XML 파일로 AOP를 구현해보았다.
그런데 스프링에서 XML 파일로 AOP를 구현하는 것보다 쉽게 구현을 할 수 있도록 관련된 어노테이션을 지원한다.
그것이 바로 @EnableAspectJAutoProxy, @Aspect, @Before, @After,
@AfterReturning, @AfterThrowing, @Around이다.

### 1. @Aspect

우리는 @Aspect를 이용해서 주요 로직에서 공통적인 관심사를 떼어 놓은 부가 기능을 구현할 수 있다.
@Aspect를 작성한 클래스의 안에는 @Before, @After, @AfterReturning, @AfterThrowing, @Around를 통해
어드바이스를 생성할 수 있다.

모든 어드바이스 어노테이션의 value 값으로는 AspectJ 표현식을 통한 메소드 지정을 할 수 있다.
그러면 그 지정한 메소드만 적용하게 설정할 수 있다.
AspectJ 표현식에 대한 자세한 설명으로는 나중에 정리하도록 하겠다.

그리고 어드바이스 어노테이션이 적용된 메소드는 매개변수로 JoinPoint 객체를 사용할 수 있는데
이 객체는 어드바이스 어노테이션의 value 요소로 들어간 포인트컷(Pointcut)표현식으로 표현한 메소드의
정보가 저장되어 있다.
JoinPoint 객체에 대한 설명도 다음에 알아보도록 하겠다.

#### 1-1. @Before

사실 XML 파일에 저장된 정보를 어노테이션으로 옮긴 것 뿐이지 사용하는 방법은 XML 방식과 거의 동일하다.
@Before를 작성하게 되면 그 메소드는 조인 포인트가 실행되기 전에 실행되는 메소드가 되는 것이다.
주로 보안 로직을 적용하거나 로그를 찍는 등의 내용이 포함된다.

```java
@Aspect
public class TestAspect {
    @Before(value = "execution(* *.*(..))")
    public void before(JoinPoint joinPoint) {
        System.out.println("@Before 실행");
    }
}
```

위의 포인트컷 표현식은 모든 메소드를 지정한다는 뜻이다.
그리고 @Before의 value 요소는 생략이 가능하기 때문에 다음과 같이 사용할 수 있다.

```java
@Aspect
public class TestAspect {
    @Before("execution(* *.*(..))")
    public void before(JoinPoint joinPoint) {
        System.out.println("@Before 실행");
    }
}
```

#### 1-2. @After

@After도 @Before과 마찬가지로 XML 파일에 존재했던 AfterAdvice 구현체와 별로 다를 것이 없다.
조인 포인트가 실행된 후 실행되는 메소드라고 볼 수 있다.
하지만 그 조인 포인트가 실행에 성공하든 성공하지 않든 무조건 실행하는 메소드라는 점이 중요하다.

```java
@Aspect
public class TestAspect {
    @Before("execution(* *.*(..))")
    public void before(JoinPoint joinPoint) { ... }
    @After("execution(* *.*(..))")
    public void after(JoinPoint joinPoint) {
        System.out.println("@After 실행");
    }
}
```

#### 1-3. @AfterReturning

@AfterReturning은 @After에서 더 자세하게 다루는 부분인데
@After는 조인 포인트 내용의 실행 성공과는 관계 없이 실행하지만,
@AfterReturning은 조인 포인트 내용이 예외 없이 성공했을 경우에만 실행하는 메소드에게 작성된다.
@AfterReturning은 value 요소가 아니라 pointcut 요소로 포인트컷을 지정한다.
그리고 @AfterReturning은 returning이라는 요소가 존재한다.
returning 요소는 String 타입의 변수 이름을 지정하면 그 변수 이름을 토대로 Object 타입의 리턴된 변수를
매개변수로 받을 수 있다.

```java
@Aspect
public class TestAspect {
    @Before("execution(* *.*(..))")
    public void before(JoinPoint joinPoint) { ... }
    @After("execution(* *.*(..))")
    public void after(JoinPoint joinPoint) { ... }
    
    @AfterReturning(
    	pointcut = "execution(* *.*(..))",
        returning = "returnValue"
    )
    public void afterReturning(JoinPoint joinPoint, Object returnValue) {
        System.out.println("@AfterReturning 실행");
    }
}
```

#### 1-4. @AfterThrowing

@AfterThrowing은 @AfterReturning이 조인 포인트(주요 로직)에서 성공적으로 return하였을 경우에만
실행하는 메소드에 작성하는 어노테이션이였다면,
@AfterThrowing은 예외 또는 에러가 발생하였을 경우에만 실행하는 메소드에 작성하는 어노테이션이다.
@AfterThrowing도 @AfterReturning와 마찬가지로 pointcut 요소를 이용해서 적용할 메소드를 선택할 수 있다.
그리고 어떤 예외 또는 에러가 발생하였을 경우에 할 것인지 예외 또는 에러 타입의 매개변수를 받을 수 있는데
그 매개변수의 이름을 지정하는 throwing 요소가 존재한다.

```java
@Aspect
public class TestAspect {
    @Before( ... )
    public void before(JoinPoint joinPoint) { ... }
    @After( ... )
    public void after(JoinPoint joinPoint) { ... }
    @AfterReturning( ... )
    public void afterReturning(JoinPoint joinPoint, Object returnValue) { ... }
    
    @AfterThrowing(
    	pointcut = "execution(* *.*(..))",
        throwing = "exception"
    )
    public void afterThrowing(JoinPoint joinPoint, Throwable exception) {
        System.out.println("@AfterThrowing 실행");
    }
}
```

Throwable 타입을 사용하게 되면 조인 포인트에서 발생하는 모든 예외 또는 에러를 받아서 처리할 수 있다.
내가 원하는 예외 또는 에러만 감지하고 싶다면 그 예외 타입을 작성하면 된다.

#### 1-5. @Around

@Around는 지금까지의 어드바이스 중 가장 강력한 기능을 가지고 있는 어드바이스이다.
@Around를 작성하게 되면 그 메소드는 Object 타입을 리턴하며 매개변수로 ProceedingJoinPoint가 된다.
ProceedingJoinPoint는 proceed() 메소드를 이용해서 조인 포인트를 실행시킬 수 있기 때문에
조인 포인트가 실행되기 전에 밑 작업을 할 수 있고 매개변수를 조작할 수 있으며
proceed() 메소드의 리턴값인 조인 포인트의 리턴값을 조작할 수도 있다.
따라서 @Around를 사용하는 것은 굉장히 위험한 도전이 될 수도 있다.
그렇기 때문에 최소한의 기능을 가진 다른 어드바이스를 사용하는 것이 좋다.

```java
@Aspect
public class TestAspect {
    @Around("execution(* *.*(..))")
    public Object around(ProceedingJoinPoint joinPoint) {
        // Before 로직
        Object result = joinPoint.proceed(); // 주요 로직 실행
        // After 로직
        return result; // 최종 결과 리턴
    }
}
```

### 2. @EnableAspectJAutoProxy

이렇게 @Aspect를 이용해서 어드바이스를 사용할 클래스를 명시하였다.
그러면 이 어드바이스를 스캔하여 사용할 @Configuration 파일에도 스캔할 수 있도록 만들어야 한다.
@Aspect가 작성된 클래스가 존재한다는 것을 인지하고 프록시 객체를 생성하는 어노테이션인
@EnableAspectJAutoProxy를 지원한다.

하지만 @Aspect를 작성한 클래스는 컴포넌트로 인식하지 않기 때문에
@Component를 작성해주어야 한다는 점이 존재한다.