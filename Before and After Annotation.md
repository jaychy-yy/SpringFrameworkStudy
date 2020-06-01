## Before and After Annotation

스프링에서 @Test를 이용하여 테스트를 진행할 때
테스트 메소드들은 하나하나씩 차례대로 실행된다.
@Test가 작성된 테스트 메소드 하나 당 객체 하나를 생성하여 실행하게 되는데
계속 반복되는 ApplicationContext와 같은 자원은 따로 빼놓고 싶을 것이다.

따라서 스프링에서는 @Test가 작성된 테스트 메소드에게 중복된 부분을
처리할 수 있도록 @Before와 @After를 지원합니다.

### 1. @Before

@Before는 작성한 메소드를 각각의 테스트 메소드가 실행되기 전에 먼저 실행합니다.
먼저 설정을 해야하는 부분이 있을 때 그 부분을 @Before를 작성한 메소드로 따로 빼서 사용하면 된다.

만약 다음과 같은 테스트 코드들이 존재한다고 하자.

```java
public class TestCodeClass {
    @Test
    public void test1() {
        List<String> list = new ArrayList<String>();
        
        list.add("aa");
        list.add("bb");
        list.add("cc");
        
        list.clear();
    }
    
    @Test
    public void test2() {
        List<String> list = new ArrayList<String>();
        
        list.add("...");
        list.add(",,,");
        list.add(";;;");
        
        list.clear();
    }
}
```

위 테스트 코드들은 먼저 List 타입의 ArrayList를 생성한다는 점에서
비슷한 점을 뽑을 수 있다.

그래서 위 테스트 코드들의 중복을 제거하기 위해서 @Before를 사용하면 다음과 같다.

```java
public class TestCodeClass {
    private List<String> list;
    
    @Before
    public void setting() {
        list = new ArrayList<String>();
    }
    
    @Test
    public void test1() {
        list.add("aa");
        list.add("bb");
        list.add("cc");
        
        list.clear();
    }
    
    @Test
    public void test2() {
        list.add("...");
        list.add(",,,");
        list.add(";;;");
        
        list.clear();
    }
}
```

현재는 테스트 코드가 짧기 때문에 간단하게 느껴질 수 있겠지만
나중에 코드량이 많아지게 되면 가독성을 높일 수 있는 방법이 될 것이다.

### 2. @After

@After는 각각의 테스트 메소드가 실행된 뒤 실행하는 메소드에 작성할 수 있습니다.
@Before의 반대 버전이라고 생각하면 된다.
위의 @Before 예제를 보면 마지막으로 list.clear() 구문이 반복된다.
이를 @After를 작성한 메소드로 변환하면 다음과 같다.

```java
public class TestCodeClass {
    private List<String> list;
    
    @Before
    public void setting() {
        list = new ArrayList<String>();
    }
    
    @Test
    public void test1() {
        list.add("aa");
        list.add("bb");
        list.add("cc");
    }
    
    @Test
    public void test2() {
        list.add("...");
        list.add(",,,");
        list.add(";;;");
    }
    
    @After
    public void clear() {
        list.clear();
    }
}
```