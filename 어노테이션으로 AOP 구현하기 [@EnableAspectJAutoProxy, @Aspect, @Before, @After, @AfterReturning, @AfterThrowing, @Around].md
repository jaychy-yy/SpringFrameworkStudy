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
이 객체는 어드바이스 어노테이션의 value 값으로 들어간 AspectJ 표현식으로 표현한 메소드의
정보가 저장되어 있다.
JoinPoint 객체에 대한 설명도 다음에 알아보도록 하겠다.

#### 1-1. @Before

사실 XML 파일에 저장된 정보를 어노테이션으로 옮긴 것 뿐이지 사용하는 방법은 XML 방식과 거의 동일하다.
@Before를 작성하게 되면 그 메소드는 조인 포인트가 실행되기 전에 실행되는 메소드가 되는 것이다.
주로 보안 로직을 적용하거나 로그를 찍는 등의 내용이 포함된다.

#### 1-2. @After

@After도 @Before과 마찬가지로 XML 파일에 존재했던 AfterAdvice 구현체와 별로 다를 것이 없다.
조인 포인트가 실행된 후 실행되는 메소드라고 볼 수 있다.
하지만 그 조인 포인트가 실행에 성공하든 성공하지 않든 무조건 실행하는 메소드라는 점이 중요하다.

#### 1-3. @AfterReturning

@AfterReturning은 @After에서 더 자세하게 다루는 부분인데
@After는 조인 포인트 내용의 실행 성공과는 관계 없이 실행하지만,
@AfterReturning은 조인 포인트 내용이 예외 없이 성공했을 경우에만 실행하는 메소드에게 작성된다.

#### 1-4. @AfterThrowing

@fter