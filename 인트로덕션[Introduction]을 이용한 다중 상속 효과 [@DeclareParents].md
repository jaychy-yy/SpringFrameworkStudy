## 인트로덕션[Introduction]을 이용한 다중 상속 효과 [@DeclareParents]

### 1. 인트로덕션(Introduction)이란?

인트로덕션이란, AOP에서 프록시 객체를 사용할 떄 프록시가 여러 인터페이스를 구현할 수 있다는 것을
이용해서 동적으로 여러 클래스를 구현하는 것처럼 표현하여 다중 상속의 효과를 낼 수 있도록 하는
기술이다.

### 2. 인트로덕션을 사용하는 이유

하나의 프록시 객체가 여러 타입으로 변환이 가능하고 그의 기능을 사용이 가능하다면
굳이 여러 개의 프록시 객체로 나눌 필요 없이 비슷한 기능을 여러 가진 프록시 객체 하나로
모든 것을 사용하는 것이 가능하다.
그래서 인트로덕션을 사용한다.

### 3. @DeclareParents를 이용한 인트로덕션 구현하기

기존의 Test 인터페이스를 구현한 TestImpl 구현 객체에 내부적으로 동적 프록시를 사용하여
사람, 늑대 인터페이스를 구현하도록 하는 @DeclareParents를 작성하면 자동으로 동적 프록시 객체를 생성하여
늑대 인터페이스와 사람 인터페이스를 등록해 그의 구현 객체를 만들어 처리를 하도록 위임한다.

이를 코드로 만들어보면 다음과 같이 만들 수 있다.

```java
package aaa.bbb.ccc;

public interface People {
    void sayYa();
}

/*
위에서 언급한 사람 인터페이스를 People 인터페이스로 만든 것이다.
*/
```

```java
package aaa.bbb.ccc;

public interface Wolf {
    void sayHow();
}

/*
위에서 언급한 늑대 인터페이스를 Wolf 인터페이스로 만든 것이다.
how는 늑대의 울음소리이다.
*/
```

```java
package aaa.bbb.ccc;

public class PeopleImpl implements People {
    public void sayYa() {
        System.out.println("ya");
    }
}

/*
동적 프록시 객체가 People 인터페이스가 처리를 넘겨줄 People 구현 객체이다.
*/
```

```java
package aaa.bbb.ccc;

public class WolfImpl implements Wolf {
    public void sayHow() {
        System.out.println("how");
    }
}

/*
동적 프록시 객체가 Wolf 인터페이스가 처리를 넘겨줄 Wolf 구현 객체이다.
*/
```

다음은 원래 인트로덕션을 적용하기 전에 컴포넌트의 역할을 하고 있었던
Test 인터페이스와 이를 구현하는 TestImpl 클래스이다.

```java
package aaa.bbb.ccc;

public interface Test {
    void test();
}
```

```java
package aaa.bbb.ccc;

@Component("test")
public class TestImpl {
    public void test() {
        System.out.println("test");
    }
}
```

@DeclareParents를 원하는 인터페이스의 필드에 작성하면 된다.
이것은 파일을 분할해놓는 것이 좋으므로 따로 파일을 나눠서 관리한다.

```java
@Aspect
@Component
public class TestIntroduction {
    @DeclareParents(
    	value = "aaa.bbb.ccc.TestImpl",
        defaultImpl = PeopleImpl.class
    )
    public People people;
    
    @DeclareParents(
    	value = "aaa.bbb.ccc.TestImpl",
        defaultImpl = WolfImpl.class
    )
    public Wolf wolf;
}
```

value는 적용할 컴포넌트의 패키지명 + 클래스명이다.
defaultImple는 사용할 인터페이스의 명을 말한다.

물론 TestIntroduction 클래스도 애스펙트 객체이기 때문에 @Aspect를 작성해야 하고 @Aspect라서
무조건 빈으로 등록되는 것이 아니기 때문에 @Component를 작성해야 한다.

시간 절약을 위해서 ApplicationContext는 생략한다.
다음은 메인 클래스이다.

```java
public class MainClass {
    public static void main(String[] args) {
        Class<IntroductionApplicationContext> iac = IntroductionApplicationContext.class;
        ApplicationContext ctx = new AnnotationConfigApplicationContext(iac);
        Test test = ctx.getBean("test");
        test.test();
        
        People people = (People) test;
        Wolf wolf = (Wolf) test;
        people.sayYa();
        wolf.sayHow();
    }
}

// test
// ya
// how
```

이렇게 test가 People과 Wolf 타입으로 변경이 가능한데 People과 Wolf가 서로 다른 기능을 가지는
기괴한 현상을 볼 수 있다.
이렇게 마치 동적으로 인터페이스를 구현한 것처럼 보이게 하는 기술이 인트로덕션이다.

### 4. 컴포넌트에 상태 추가하기

POJO 즉, 컴포넌트에 상태를 추가한다는 것은 딱히 지금까지 배웠던 것과 다른 개념이 아니다.
컴포넌트에 상태를 추가할 때 모든 컴포넌트에게 인터페이스를 구현시켜서 적용시키는 것은
동일한 기능을 모두에게 상속시키는 것이므로 적절치 못합니다.
따라서 스프링에서는 프록시 객체 즉, AOP 기술을 이용하여 컴포넌트에 상태를 추가할 수 있도록 합니다.

만약 위에서 만들었던 POJO(컴포넌트)인 test 빈에게 test를 출력할 때마다 count를 증가시키는
기능을 넣고자 할 때 우리는 TestImpl 컴포넌트에게 새로운 인터페이스를 구현하도록 하는 것 보다는
동적 프록시 객체를 이용하여 모든 컴포넌트가 기능을 수행하도록 하는 것이 좋다.

그래서 우리는 Counter라는 인터페이스를 선언하고 CounterImpl이라는 구현체를 만들어
프록시 객체로 생성합니다.

```java
package abc.def.ghi;

public interface Counter {
    void plus();
    int getCount();
}
```

```java
package abc.def.ghi;

public class CounterImpl implements Counter {
    private int count;
    
    public void plus() {
        this.count++;
    }
    
    public int getCount() {
        return count;
    }
}
```

그리고 IntroductionTest 클래스에 @DeclareParents를 추가하고
@After를 작성하여 test() 메소드를 실행시킬 때마다 count 값을 1씩 올립니다.

```java
package abc.def.ghi;

public class IntroductionTest {
    @DeclareParents(
    	value = "abc.def.ghi.*Test*",
        defaultImpl = CounterImple.class
    )
    public Counter counter;
    
    @After("execution(* *.*(..)) && this(counter)")
    public void printCount(Counter counter) {
        counter.plus();
    }
}
```

그리고 메인 클래스에서 Test 객체를 Counter 타입으로 변경하여 getCount() 메소드를 사용하면
자신이 test() 메소드를 몇 번 호출했는지 알 수 있다.