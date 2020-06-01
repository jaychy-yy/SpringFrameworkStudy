## BeforeClass and AfterClass Annotation

이번에는 @BeforeClass와 @AfterClass에 대해서 알아보도록 하겠다.
저번에 @Before와 @After에 대해서 공부할 때
@Before는 @Test 테스트 메소드가 실행되기 전에 실행하는 메소드에 작성하고,
@After는 @Test 테스트 메소드가 실행된 후에 실행하는 메소드라고 했었다.

하지만 @Before 또는 @After가 작성된 메소드는 각각의 테스트 코드가 실행될 때
모두 실행되기 때문에 여러 번 실행되는 일이 발생할 수 있다.
그렇게 되면 자원의 중복이 일어날 수 있기 때문에 되도록이면 피하는 것이 좋다.

따라서 이를 막기 위해 @BeforeClass와 @AfterClass를 제공한다.

### 1. @BeforeClass

@BeforeClass는 모든 테스트 코드가 실행되기 전 딱 한 번만 실행되는 메소드에 작성한다.
따라서 ApplicationContext와 같이 무거운 자원을 여러 번 생성하는 일은 없을 것이다.

```java
public class TestCodeClass {
    private static ApplicationContext ctx;
    
    @BeforeClass
    public void beforeMethod() {
        ctx = new GenericXmlApplicationContext("applicationContext.xml");
    }
    
    public void test1() {
        List<String> list = ctx.getBean("list", List.class);
        ...
    }
    
    public void test2() {
        List<String> list = ctx.getBean("list", List.class);
        ...
    }
}
```

이렇게 static 필드에 할당하여 사용할 수 있다.

### 2. @AfterClass

@AfterClass는 @BeforeClass와는 반대로 모든 테스트 메소드가 종료되면 실행하는 메소드이다.

### 3. Spring에서 지원하는 스프링 테스트 컨텍스트

스프링에서는 위와 같은 @Before, @AfterClass 대신 JUnit의 확장을 통한 테스트를 지원한다.
사용해야하는 어노테이션으로는 @RunWith, @ContextConfiguration, @Autowired가 있다.

#### 3-1. @RunWith

@RunWith는 JUnit의 기능을 확장하여 사용하기 위해서 사용하며
요소로 SpringJUnit4ClassRunner라는 클래스 변수를 넣어주면 된다.

#### 3-2. @ContextConfiguration

@ContextConfiguration은 자동으로 생성할 ApplicationContext의 경로를 설정한다.
이것으로 이 어노테이션들은 ApplicationContext를 설정하기 위한 어노테이션이라는 것을 알 수 있다.

#### 3-3. @Autowired

@Autowired는 전에 의존성 자동 주입에서 다뤘던 어노테이션으로서,
Autowired Annotation 글을 봤다면 알 수 있다.

이렇게 사용하는 방법은 다음과 같다.

```java
@RunWith(SpringUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml");
public class TestCodeClass {
    @Autowired
    private ApplicationContext ctx;
    
    public void test() {...}
}
```