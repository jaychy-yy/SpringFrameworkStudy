## org.thymeleaf.exceptions.TemplateInputException

### 1. 개요

org.thymeleaf.exceptions.TemplateInputException이 발생하여
스프링 부트가 실행되지 못하고 있다.

### 2. 발생 이유

TemplateInputException의 이름으로 유추할 수 있듯이 템플릿을 사용하다가 잘못된 인풋으로 인해서
예외가 발생한 것으로 유추할 수 있다.

TemplateInputException은 org.thymeleaf.exceptions 패키지에 존재하는 예외이므로
Thymeleaf 템플릿 엔진을 사용하다가 발생한 예외라는 것을 알 수 있다.
그런데 아무리 찾아봐도 Thymeleaf 템플릿 엔진을 사용한 코드에는 문제가 없다는 것을 알게 되었고,
구글링을 통해 원인을 알게 되었다.

### 3. 해결

결과는 Controller(컨트롤러)에서 문제를 일으킨 것으로 확정되었다.
컨트롤러의 리턴 부분, 즉 어느 템플릿 엔진을 사용할 것인지에 대한 주소를 결정할 때
처음에 '/'를 통한 루트 경로 입력을 했기 때문에 예외가 발생한 것이었다.
만약 다음과 같이 컨트롤러 코드가 존재한다면 예외가 발생한다.

```java
@Controller
public class TestController {
    
    @GetMapping("/")
    public String home() {
        return "/home";
    }
}
```

이러면 resources 아래의 templates 아래의 home.html을 찾아갈 것만 같지만
/home 경로로 가기 때문에 resources, templates를 거치지 않고 루트 경로로 인식하여
home.html을 찾을 수 없다는 예외가 발생하게 된다.

따라서 이를 해결하기 위해서는 주소의 앞 부분에 '/'(루트 경로)를 제외해주면 정상적으로 동작한다.

```java
@Controller
public class TestController {
    
    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```

