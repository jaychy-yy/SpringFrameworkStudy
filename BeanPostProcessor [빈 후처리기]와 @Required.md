## BeanPostProcessor [빈 후처리기]와 @Required

### 1. 빈 후처리기란?

빈 후처리기(Bean Post Processor)란 빈 후처리(Bean Post Processing)을 진행하는 객체를 말합니다.
빈 후처리란 실제 빈 초기화 -> initMethod -> 사용의 형태로 이루어지는 빈의 생명주기에서
initMethod가 실행되기 전과 실행된 후에 후처리를 진행할 수 있는 메소드를 말합니다.

### 2. BeanPostProcessor를 이용한 빈 후처리기 설정하기

이렇게 빈 후처리를 진행할 수 있는 메소드는 BeanPostProcessor라는 인터페이스를 이용해서 구현할 수 있는데,
BeanPostProcessor 인터페이스를 구현한 객체를 postProcessBeforeInitalization() 메소드와
postProcessAfterInitialization() 메소드를 구현해야 합니다.

postProcessBeforeInitialization() 메소드는 initMethod가 실행되기 전에 실행이 되며,
postProcessAfterInitialization() 메소드는 initMethod가 실행된 후에 실행이 됩니다.
두 메소드는 후처리를 진행할 객체를 Object 타입, bean이라는 이름으로 받고
두 번째 매개변수로 String 타입의 beanName을 받습니다.
이를 이용해 후처리를 적용한 빈의 내용을 변경하고 싶다면 Object타입의 bean 변수를 조작하면 됩니다.

BeanPostProcessor 인터페이스를 구현한 MyBeanPostProcessor 클래스를 작성해봅시다.

```java
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        System.out.println("initMethod 실행 전에 실행할 메소드");
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, Strinb beanName) {
        System.out.println("initMethod 실행 후에 실행할 메소드");
    }
}
```

이미 클래스는 인스턴스화되어 객체가 생성되었기 때문에 bean을 건드려서 객체를 변경시켜도 된다.
위 빈 후처리기는 @Component를 이용해 빈으로 등록되었는데 이러면 자동으로 스프링 컨테이너는
@Component가 작성된 BeanPostProcessor의 구현체를 빈으로 등록하여 모든 빈들에게 적용한다.

### 2. 일부 빈(Bean)들만 빈 후처리기 설정하기

위의 예제처럼 빈 후처리기를 빈으로 등록하면 모든 빈들에게 빈 후처리 과정이 적용된다.
이러면 원하지 않은 결과를 초래할 수 있기 때문에
instanceof 연산자를 사용해서 원하는 타입의 빈에만 적용할 수 있도록 할 수 있다.
또한 beanName은 빈의 이름이 String 타입으로 저장되어 있기 때문에 이름으로 구별하기도 가능하다.

```java
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        if(bean instanceof Student) {
            System.out.println("Student 타입의 객체");
       	 	System.out.println("initMethod 실행 전에 실행할 메소드");
        }
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, Strinb beanName) {
        if(bean instanceof Student) {
            System.out.println("Student 타입의 객체");
        	System.out.println("initMethod 실행 후에 실행할 메소드");
        }
    }
}
```

### 3. @Required를 이용한 빈 후처리기 설정하기

사실 이 빈 후처리기는 매개변수가 잘못되지는 않았는지 값이 들어는 왔는지에 대해서 알아보는데 사용된다.
따라서 이 능력을 쉽게 사용하고자 @Required라는 어노테이션이 나오게 되었는데,
@Required이 작성된 메소드는 그 메소드가 존재하는 빈이 스프링 컨테이너에 등록되었을 때
정상적으로 DI가 되었는지 확인하고 만약 DI가 이루어지지 않아 값이 등록되지 않았다면
BeanInitializationException을 발생시킵니다.
참고로 @Required는 내부적으로 RequiredAnnotationBeanPostProcessor를 사용합니다.