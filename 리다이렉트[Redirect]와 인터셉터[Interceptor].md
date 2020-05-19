## 리다이렉트[Redirect]와 인터셉터[Interceptor]

오늘은 리다이렉트와 인터셉터에 대해서 알아보았다.

목차는 간단하다.

1. 리다이렉트[Redirect]
2. 인터셉터[Interceptor]



### 1. 리다이렉트[Redirect]

#### 1-1. 정의

리다이렉트는 Controller에서 View를 분기하는 기술을 말한다.

#### 1-2. 사용 이유

main.jsp에서 modify.jsp로 간다고 했을 때 만약 URL을 조작해서 로그인을 하지 않고
modify.jsp로 갔다고 가정해보자.

그러면 정상적인 경로가 아닌 다른 경로로 들어왔으니
세션에 로그인 정보가 없거나 쿠키에 로그인 정보가 없거나,
정상적인 로그인 정보가 아닐 것이다.
그러므로 이를 확인하여 다시 main 페이지 또는 login 페이지로 보낼 수 있다.
이렇게 main.jsp, login.jsp이나 View를 분기하기 위해서 리다이렉트를 사용한다.

#### 1-3. 사용 방법

만약 위와 같은 상황을 예로 들어 @RequestMapping("/user/login")을 작성한 메소드를 살펴보자.

```java
@Controller
public class UserController {
    @RequestMapping("/user/login")
    public String modify(Model model, HttpServletRequest request) {
        HttpSession session = request.getSession();
        User user = session.getAttribute("user");
        
        if(user == null) {
            return "redirect:/login";
        } else {
            // Modify 하는 부분
        }
        return "user/modifyOk";
    }
}
```

이렇게 작성하게 되면 로그인 정보를 세션에 담아두었다는 가정하에
세션의 로그인 정보를 가져오고 만약 로그인 정보가 없다면(null 이라면) 리다이렉트하여
로그인 페이지로 돌려보낸다.

만약 아니라면 정상적인 작동을 하게 된다.

위를 보면 알겠지만 리다이렉트를 사용하는 방법은 redirect:경로이다.

#### 1-4. 의문점

그러면 리다이렉트를 하지 않고 그냥 return "경로"를 통해 View로 이동하면 되는 게 아닌가요?

이런 의문이 들 수 있다.
하지만 리다이렉트를 이용해 로그인 페이지로 넘어가는 것과 로그인 페이지 View로 넘기는 것이 조금 다르다.

로그인 페이지 View로 바로 보내는 일반적인 리턴방식은 원래 로그인 페이지로 넘어갔던
Controller를 거치지 않는다.
하지만 리다이렉트를 하게 되면 로그인 페이지로 넘기는 Controller의 메소드를 실행하게 된다.
따라서 그냥 리턴하는 것은 컨트롤러를 더 이상 거치지 않고 페이지를 옮긴다면,
리다이렉트는 컨트롤러를 거치고 페이지를 옮기게 된다.

### 2. 인터셉터[Interceptor]

#### 2-1. 정의

Controller가 작동하지 전, 작동한 후, View에 보여진 후에 작동하는 코드를 설정하는 기술이다.

#### 2-2. 사용 이유

리다이렉트를 사용할 때 우리는 로그인이 되었는지를 확인하는 코드를 작성했었다.
그런데 로그인이 되어 있어야 하는 작업의 페이지가 많다면 일일이 그 작업을 해주어야할까?
이렇게 많은 작업을 단축하기 위해서 인터셉터를 사용한다.

#### 2-3. 사용 방법

인터셉터는 HandlerInterceptor 인터페이스를 구현하여 사용할 수 있다.
이를 구현하는 클래스는 Controller가 아니라 일반 클래스이며
HandlerInterceptor는 인터페이스이기 때문에 모두 사용하지는 않더라도
세 메소드를 모두 구현해야 한다.
구현해야하는 세 메소드는 다음과 같다.

```java
public boolean preHandle(HttpServletRequest request,
                         HttpServlet response, Object handler);
public void postHandle(HttpServletRequest request,
                       HttpServlet response, Object handler,
                       ModelAndView modelAndView);
public void afterCompletion(HttpServletRequest request,
                      		HttpServlet response, Object handler,
                            Exception ex);
```

preHandle() 메소드는 Controller의 메소드가 실행되기 전에 실행되는 메소드이고,
postHandle() 메소드는 Controller의 메소드가 실행된 후에 실행되는 메소드이고,
afterCompletion() 메소드는 Controller가 잘 동작하여 View에 값이 전달되어 View가 출력되었을 때
동작하는 메소드이다.

여담으로 preHandle() 메소드가 가장 많이 쓰이고, afterCompletion() 메소드는 잘 사용하지 않는다고 한다.

위의 리다이렉트 예제처럼 로그인을 이용한 페이지에서
로그인을 하였는지 세션 체크를 하기 위해서는 preHandle() 메소드에
아래와 같이 코드를 작성하면 된다.

```java
public class TestHandler implements HandlerInterceptor {
    public boolean preHandle(HttpServletRequest request,
                         HttpServlet response, Object handler) {
        System.out.println("컨트롤러 메소드 실행 전 출력");
        
        HttpSession session = request.getSession();
        if(session != null) {
            User user = session.getAttribute("user");
            if(user != null) {
                return true;
            }
        }
        response.sendRedirect(request.getContextPath() + "/login");
        return false;
    }
    public void postHandle(HttpServletRequest request,
                           HttpServlet response, Object handler,
                           ModelAndView modelAndView) {
		System.out.println("컨트롤러 메소드 종료 후 출력");
    }

	public void afterCompletion(HttpServletRequest request,
                      		HttpServlet response, Object handler,
                            Exception ex) {
        System.out.println("뷰에 정상적인 출력");
    }
}
```

하지만 이렇게 인터페이스로 구현하면
preHandle() 메소드만 사용하고 싶은데 postHandle(), afterCompletion() 메소드를 모두 구현해야하는
단점이 생기게 된다.

그래서 스프링에서는 HandlerInterceptorAdapter 클래스를 사용한다.
이 클래스를 상속받아서 사용하고 싶은 메소드만 구현하면 정상적으로 원하는 기능을 사용할 수 있다.

여기서 특히 주요로 봐야하는 게 preHandle() 메소드이다.
preHandle() 메소드는 컨트롤러 메소드가 작동되기 전에 작동되는 메소드라고 했다.
그런데 preHandle() 메소드는 리턴값으로 boolean 타입을 가진다.
여기서 true를 리턴하게 되면 정상적으로 컨트롤러가 실행되지만
false를 리턴하게 되면 postHandle(), afterCompletion() 메소드는 물론 실행이 되지 않고
컨트롤러도 실행되지 않게 된다.