## Life Cycle [생명 주기]

이번에는 스프링 컨테이너의 생명 주기에 대해서 공부해보았다.
스프링 컨테이너는 자바 빈을 포함하고 있고, 지금은 자바 빈만을 다룬다.
나중에 더 알게되면 추가해보도록 하겠다.

스프링 컨테이너의 생명 주기는 크게 세 가지로 나누어진다.

1. 초기화
2. 사용
3. 종료

#### 1. 초기화

우리가 스프링 설정 파일을 구성하고 GenericXmlApplicationContext 클래스를 이용하여
스프링 컨테이너를 생성할 수 있다.
이 때 스프링 컨테이너 안의 모든 빈 객체들이 메모리에 할당되고 getBean() 메소드를 통해
객체를 사용할 수 있는 상태가 된다.
이 상태를 우리는 스프링 컨테이너의 초기화라고 한다.

#### 2. 사용

스프링 컨테이너를 사용한다는 것은 스프링 컨테이너 안의 빈 객체들을 사용한다는 뜻이다.
따라서 getBean() 메소드를 통해 객체를 받아서 사용하고 이것저것 사용하는
이 상태가 가장 많이 볼 수 있는 사용상태이다.

#### 3. 종료

스프링 컨테이너의 모든 빈 객체들을 사용하고 이젠 사용할 일이 없다고 판단하였을 때
개발자는 코드에서 close() 메소드를 사용함으로서 GenericXmlApplicationContext 객체를 소멸시킨다.
이 때, 스프링 컨테이너가 종료됨과 동시에 안에 있던 빈 객체들도 모두 소멸되게 된다.
이 상태가 스프링 컨테이너의 종료상태이다.



이렇게 스프링 컨테이너는 크게 세 가지로 나누어지는 생명 주기를 가지고 있다.
처음엔 초기화를 하고 사용되다가 종료되면 소멸된다.

이 얘기를 한 이유는 스프링 컨테이너가 초기화 되고 사라지는 시점에 빈 객체들도 초기화 되고 사라진다.
따라서 객체들이 초기화되고 소멸되는 시점에 실행할 수 있는 메소드가 존재한다.
이런 메소드를 만드는 방법에는 크게 두 가지가 있다.
InitializingBean 인터페이스와 DisposableBean 인터페이스를 사용하는 방법과
init-method 속성과 destroy-method 속성을 사용하는 방법이 있다.

### InitializingBean - DisposableBean

InitializingBean과 DisposableBean은 위에서 말했듯 인터페이스로,
원하는 클래스에 구현시켜서 객체가 초기화 될 때와 소멸 될 때 작동하는 메소드를 만들 수 있다.
InitializingBean 인터페이스는 afterPropertiesSet() 메소드를 구현해야하고
DisposableBean은 destroy() 메소드를 구현해야 한다.
afterPropertiesSet() 메소드는 객체가 스프링 컨테이너에 생성될 때 실행되는 메소드이고
destroy() 메소드는 객체가 close() 메소드와 동시에 소멸될 때 실행되는 메소드이다.

두 메소드의 형식은 다음과 같다.

```java
// afterPropertiesSet
public void afterPropertiesSet() throws Exception {
    // 객체 초기화시 실행 내용
}

// destroy
public void destroy() throws Exception {
    // 객체 소멸시 실행 내용
}
```

꼭 두 가지를 동시에 구현할 필요는 없고 하나만 구현해도 된다.
따라서 다음과 같이 