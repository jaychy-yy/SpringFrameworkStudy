## @Profile

### 1. @Profile의 배경

스프링에서는 3.1 버전 이후로 @Profile이라는 어노테이션을 지원한다.
@Profile은 쉽게 생각하면 스프링 컨테이너에 등록될 빈을 설정하는 것이고,
어떤 구성으로 작업할지를 대비하기 위해서 빈의 그룹을 만드는 것이라고 생각할 수도 있다.

우리가 @Profile을 사용하는 이유는 흔히 회사에서 하는 개발 단계에 있다.
회사에서는 먼저 기획을 하고 개발을 한 뒤 테스트를 진행하고 유지보수하며 운영을 시작한다.
먼저 기획부분은 빼고 개발 -> 테스트 -> 실제 서비스(유지보수)라고 했을 때
개발 및 테스트하면서 사용하는 자원이 있을 것이고 실제 서비스를 할 때 필요한 자원이 있을 것이다.

가장 쉽게 떠오르는 것이 DB 설정이다.
만약 실제로 서비스되고 있는 애플리케이션의 DB를 이용해서 테스트를 하게 된다면
실수를 했을 때의 문제가 엄청나게 커질 것이다.
그래서 흔히들 테스트용 DB와 실제 서비스용 DB를 나누어 놓는다.
그러면 테스트용 DB를 사용하는 코드와 서비스용 DB를 사용하는 코드를 따로 만들어야 하는데
그러면 사용할 때마다 사용하지 않는 코드의 @Bean태그를 제거하거나,
주석 처리하거나, @Component태그를 제거하는 경우가 있다.
하지만 이럴 경우 사용하지 않는 코드들을 모두 제거/주석해야한다는 문제가 생기고
만약 DB1을 사용하기 위해서 DB2 ... DB100까지 주석처리를 해야한다면?
당연히 문제가 생길 것이고 개발자 입장에서는 불만이 생길 것이다.
따라서 나온 것이 @Profile이다.

### 2. @Profile 사용하기

@Profile은 위에서 말했듯 등록될 빈을 설정하는 것이라고 했다.
@Profile의 value 요소에 String 타입으로 사용할 프로필 이름을 설정하면
ApplicationContext(스프링 설정파일)에서 사용할 프로필을 설정할 수 있다.

다음은 테스트용 DB를 사용하는 코드와 서비스용 DB를 사용하는 코드를 @Profile을 통해서 나눠놓은 코드이다.

```java
@Component
@Profile("testdb")
public class UseTestDB { /* 테스트용 DB를 사용하는 코드 */ }
```

```java
@Component
@Profile("servicedb")
public class UseServiceDB { /* 서비스용 DB를 사용하는 코드 */ }
```

지금은 설정할 Profile이 하나라서 문자열로 표현하였지만 CSV 형태로 나타내도 된다.

```java
@Profile({"servicedb"})
public class UseServiceDB { ... }
```

그럼 메인 클래스에서 ApplicationContext를 사용할 경우
이 @Profile의 설정에 따라 빈을 사용할지 말지를 결정할 수 있다.
다음은 testdb 프로필이 설정된 빈을 사용하는 메인 클래스 코드이다.

```java
public class MainClass {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext();
        Environment environment = ctx.getEnvironment();
        environment.setActiveProfiles("testdb");
        ctx.scan("test.test.test");
        ctx.refresh();
    }
}
```

스프링 설정파일의 프로필을 변경하기 위해서는 스프링 설정파일을 담당하는 ApplicationContext의
getEnvironment() 메소드를 통해 스프링 컨테이너의 환경 설정을 담당하는 Environment 객체를 뽑아낸다.
그 다음 setActiveProfiles() 메소드를 통해 여러 프로필의 이름을 String 타입으로 받는다.
그리고 타입이 String ... 타입으로 가변인자이기 때문에 여러 프로필의 이름을 가질 수 있다.
scan과 refresh를 통해 결과를 확정시켜야 정상적으로 작동하므로 꼭 해주어야 한다.
따라서 UseTestDB 클래스의 패키지는 test.test.test라는 것도 알 수 있었다.
이렇게 테스트 코드와 실제 서비스 코드를 나눌 수 있는 @Profile에 대해서 알아보았다.