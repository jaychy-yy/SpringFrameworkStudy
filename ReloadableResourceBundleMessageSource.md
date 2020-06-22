## ReloadableResourceBundleMessageSource

ReloadableResourceBundleMessageSource 클래스는 MessageSource 인터페이스를 구현한 구현체이다.
MessageSource는 각각의 언어로 된 파일을 읽어 사용할 수 있게 도와주는 인터페이스이다.
따라서 ReloadableResourceBundleMessageSource 클래스는 각각의 언어로 된 properties 파일을
각각의 나라 상황에 맞는 언어로 보여주기 위해서 존재하는 클래스이다.

ReloadableReousrceBundleMessageSource 객체를 생성할 때 Spring에서는 쉽게 객체(빈)를 생성할 수 있도록
messageSource라는 이름의 빈이 있다면 자동으로 스프링 컨텍스트에 추가하도록 구성되어 있습니다.
따라서 이 이름에 맞도록 MessageSource를 작성하면 원하는대로 각각의 언어로 구성된
properties 파일을 사용할 수 있습니다.

다음은 MessageSource 객체를 생성하기 위한 Configuration 파일이다.

```java
@Configuration
public class MessageSourceConfiguration {
    @Bean
    public MessageSource messageSource() {
        ReloadableResourceBundleMessageSource messageSource =
            new ReloadableResourceBundleMessageSource();
        messageSource.setBasenames("classpath:messages");
        messageSource.setDefaultEncoding("UTF-8");
        messageSource.setCacheSeconds(1);
        return messageSource;
    }
}
```

이렇게 Configuration 파일을 구성하면 ReloadableResourceBundleMessageSource 객체를 생성하는
빈을 구성하 수 있게 된다.
그리고 이 빈을 이용해 메인 클래스에서 getMessage() 메소드를 호출하게 되면
세 가지의 매개변수와 함께 그에 맞는 언어로 된 문장이 구성되게 된다.

그럼 메인 클래스를 구성하기 전에 Properties 파일을 구성해야 하는데
MessageSource 빈의 이름이 정해져 있었던 것처럼 가져오는 Properties 파일의 이름도 정해져 있다.
만약 미국에서 사용하는 언어로 구성되어 있는 Properties 파일의 이름은
messages_en_US.properties이다.
만약 미국 로케일로 호출하였는데 messages_en_US.properties 파일이 존재하지 않는다면
messages_en.properties 파일을 찾아서 로드하고 만약 이것도 없다면
없을 경우에 사용하는 Properties 파일인 messages.properties 파일을 로드하게 된다.
한국에서 사용하는 한글을 통한 Properties 파일은 messages_ko_KR.properties 파일이다.

따라서 미국에서 사용하는 영어로 이루어진 Properties 파일은 messages_en_US.properties 파일이므로
messages_en_US.properties 파일을 구성해보겠다.

```properties
# messages_en_US.properties
message.english.data1 = Hello, My name is JinHyeok Lee
message.english.data2 = Hello, My name is {0}
```

Properties 파일에 {0}과 같이 format 형식을 해놓으면 MessageSource 객체를 getMessage() 메소드로
호출할 때 값을 저장할 수 있다.
그럼 메인 클래스를 구성해보겠다.

```java
public class MainClass {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext();
        String m1 = ctx.getMessage("message.english.data1", null, Locale.US);
        String m2 = ctx.getMessage("message.english.data2",
                       new Object[] {"Lee Jin Hyeok"}, Locale.US);
        System.out.println(m1);
        System.out.println(m2);
    }
}

// Hello, My name is JinHyeok Lee
// Hello, My name is Lee Jin Hyeok
```

첫 번째 매개변수는 Properties 파일에 있는 key(키)값을 받고,
두 번째 매개변수는 {0}과 같이 Format String 형식으로 되어 있는 곳에 넣을 인자를 Object[] 타입으로 받고,
세 번째 매개변수는 어떤 나라의 언어 형식으로 파일을 정할 건지 Locale 객체를 받는다.