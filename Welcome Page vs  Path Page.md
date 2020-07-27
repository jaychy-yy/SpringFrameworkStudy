## Welcome Page vs / Path Page

### 1. 개요

Welcome Page라고 한다면 스프링에서는 index.html을 말한다.
index.html은 다른 언어에서도 딱히 다른 변경을 가하지 않으면 루트 경로(/)로 들어오는 요청을
받아들여서 보여주는 html이라고 알고있다.

물론 스프링에서도 index.html이 Welcome Page 역할을 진행한다.
하지만 Controller에서 / 요청으로 들어오면 다른 페이지로 넘어가도록 설정하게 되면 어떻게 될까?

### 2. 결론

딱히 별 거 없이 우선순위를 기준으로 결정된다.
/ Path Page > Welcome Page의 우선순위를 가지고 있다.
따라서 Controller에서 / Path Page를 설정해두었다면 / Path Page가 작동하게 되고
Welcome Page는 무시되며 다른 충돌은 존재하지 않는다.

### 3. 실습

인텔리제이에서 제공하는 resources 파일의 static 파일에 존재하는 Welcome Page 즉, index.html을 생성한다.

```html
<!DOCTYPE HTML>
<html>
<body>
    <h1>Welcome Page</h1>
</body>
</html>
```

그리고 resources 파일의 templates 파일에 helloworld 파일을 생성한다.

```html
<!DOCTYPE HTML>
<html>
<body>
    <h1>/ Path Page</h1>
</body>
</html>
```

그리고 / 경로를 지정하는 페이지를 작동시키는 Controller(컨트롤러)를 생성한다.

```java
@Controller
public class PathController {
    
    @GetMapping("/")
    public String helloWorld() {
        return "helloworld";
    }
    
}
```

이 상태로 스프링 부트를 실행시키면 helloworld 파일이 출력되는 것을 볼 수 있다.