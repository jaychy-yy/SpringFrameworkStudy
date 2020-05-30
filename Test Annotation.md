## Test Annotation

이번에는 @Test에 대해서 알아보도록 하겠다.
@Test에 대한 목차는 다음과 같다.

1. @Test가 무엇인가?
2. @Test는 왜 사용하는가?
3. @Test를 사용하는 방법
4. @Test와 스프링의 관계



### 1. @Test가 무엇인가?

먼저 우리는 @Test, Test 어노테이션이 무엇인가에 대해서 알아야 한다.
@Test는 이름 그대로 테스트를 하는데 사용되는 어노테이션이다.

@Test는 org.junit 패키지에 존재하며 import할 때 다음과 같이 사용된다.

```java
import org.junit.Test;
```

따라서 @Test는 스프링이 아니라 JUnit이라는 프레임워크에서 지원하는 어노테이션이라는 것을 알 수 있다.

### 2. @Test는 왜 사용하는가?

@Test는 테스트를 보다 쉽게 하기 위해서 사용된다.
우리가 @Test를 사용하지 않고 테스트를 하기 위해서는 if문을 이용해서 조건 검사를 해보거나,
실행하는 도중에 예외가 발생하면 문제가 생겼다는 것을 인지하고 에러문을 고치게 된다.
하지만 JUnit에서 지원하는 @Test를 사용하게 되면 보다 쉽게 코드를 테스트하고 검증하는 절차를 치룰 수 있다.

만약 다음과 같은 코드가 있다고 하자.
DataSource를 자동 주입하는 스프링 설정 파일은 생략하였고,
코드가 너무 길어지기 때문에 간단한 User 클래스를 생략하였다.

```java
public class UserDao {
    private DataSource dataSource;
    
    public UserDao(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    
    public void insert(User user) {
        Connection conn = null;
        PreparedStatement pstmt = null;
        String sql = "INSERT INTO users(id, pw, name) values(?,?,?)";
        
        try {
            conn = dataSource.getConnection();
            pstmt = conn.prepareStatement(sql);
            pstmt.setString(1, user.getId());
            pstmt.setString(2, user.getPw());
            pstmt.setString(3, user.getName());
            pstmt.executeUpdate();
        } catch(Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if(pstmt != null) pstmt.close();
                if(conn != null) conn.close();
            } catch(Exception e) {
                e.printStackTrace();
            }
        }
    }
    
    public User select(String id) {
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        String sql = "SELECT * FROM users WHERE id = ?";
        User user = null;
        
        try {
            conn = dataSource.getConnection();
            pstmt = conn.prepareStatement(sql);
            pstmt.setString(1, id);
            rs = pstmt.executeQuery();
            
           	if(rs.next) {
                user = new User();
                user.setId(rs.getString("id"));
                user.setPw(rs.getString("pw"));
                user.setName(rs.getString("name"));
            }
        } catch(Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if(rs != null) rs.close();
                if(pstmt != null) pstmt.close();
                if(conn != null) conn.close();
            } catch(Exception e) {
                e.printStackTrace();
            }
        }
        
        return user;
    }
}
```

이런 데이터베이스에 정보를 저장하고 검색할 수 있는 UserDao라는 클래스가 만들어졌다.
그러면 이 UserDao라는 클래스를 테스트하기 위해서 이를 동작하기 위한
HTML, CSS, JS는 물론이고 그에 따른 Controller, 다른 서비스, View 단을 만들어야 한다.
하지만 이렇게 만들어서 테스트를 할 경우
HTML, JS 소스의 문제였는지,
아니면 Controller가 원하는 URL로 이동을 시켜주지 않아서였는지,
아니면 다른 서비스의 문제였는지,
아니면 View단에서 오류가 발생해 실제로 코드에는 문제가 없는데 문제가 있는 것처럼 보였는지
알 수 없게 된다.

그리고 이렇게 복잡한 로직을 저 UserDao 클래스를 위해 작업하는 것은 매우 고되고 힘든 일이다.
따라서 이런 복잡한 로직을 구현하는 것은 큰 리스트가 따르게 되므로 사용하지 않는 것이 좋다.

그렇다면 어떻게 테스트를 해야할까?
대부분은 자바에서 지원하는 main() 메소드를 이용해서 테스트를 진행할 것이다.
만약 main() 메소드를 작성한다면 다음과 같이 작성할 수 있다.

```java
public class UserDaoTest {
    public static void main(String[] args) {
        String path = "applicationContext.xml";
        ApplicationContext ctx = new GenericXmlApplicationContext(path);
        
        UserDao userDao = ctx.getBean("userDao", UserDao.class);
        User user = new User();
        user.setId("id");
        user.setPw("pw");
        user.setName("name");
        
        userDao.insert(user);
        
        User testUser = userDao.select("id");
        if(!(testUser.getPw().equals(user.getPw())) {
            System.out.println("PW가 잘못되었습니다.");
        } else if(!(testUser.getName().equals(user.getName()))) {
            System.out.println("NAME가 잘못되었습니다.");
        }
           
        System.out.println("테스트가 성공적으로 마쳤습니다.");
    }
}
```

이렇게 main 메소드를 작성하여 테스트를 하는 것도 좋지만,
main 메소드를 이용하여 테스트를 할 경우에는 그에 맞는 예외 처리를 통해
그에 맞는 예외 출력문을 작성해야 한다.

하지만 JUnit을 이용한 @Test를 사용하면 간단한 비교를 통해 오류가 발생하였는지 체크하고
문제가 생기지 않았다면 문제가 생기지 않았고 몇 개의 테스를 했는지 알려주며
만약 문제가 생겼을 경우 무슨 테스트 코드에서 예외가 발생했는지 알 수 있다.
따라서 JUnit의 @Test를 사용하는 것이 좋다.

### 3. @Test를 사용하는 방법

@Test는 메소드에 작성할 수 있는 어노테이션이다.
하지만 특정한 조건을 만족하는 어노테이션이여야 하는데,
@Test를 작성한 메소드는 접근 제어 지시자가 반드시 public이여야 하며,
반환형이 void여야 하고, 매개변수가 없어야 한다.

위 조건을 만족하였을 때 @Test를 테스트 메소드에 작성하게 되면
그 안에 assertThat()와 같은 메소드를 사용할 수 있게 된다.
assertThat() 메소드는 static으로 import 하였기 때문에 별도의 객체를 생성하지도,
그 객체와의 관계를 연결하지도 않고 그냥 사용할 수 있다.

assertThat() 메소드의 안에 사용되는 is() 메소드도 마찬가지이다.
assertThat() 메소드를 사용하는 방법은 다음과 같다.

#### 3-1. assertThat() 메소드 사용하기

assertThat() 메소드는 기존의 main() 메소드로 테스트할 때의 if문과 같은 역할을 한다.
첫 번째 매개변수로 들어오는 값과 두 번째 매개변수, is() 메소드를 통한 값이 같을 경우
JUnit은 테스트가 끝났을 때 성공한 assertThat() 메소드의 갯수와 만약 틀렸을 경우
틀린 갯수와 그 틀린 assertThat() 메소드를 알려준다.

#### 3-2. is() 메소드 사용하기

assertThat() 메소드의 두 번째 매개변수로 is() 메소드의 값이 들어가게 되는데
is() 메소드의 값은 그냥 두 번째 매개변수로 들어가는 값을 그냥 넣으면 된다.
별 다를 것은 없다.



그럼 위의 main() 메소드를 통한 테스트 방식에서 JUnit의 @Test를 이용한 테스트 방식으로 변경해보겠다.

```java
public class UserDaoTest {
    @Test
    public void insertAndSelect() {
        String path = "applicationContext.xml";
        ApplicationContext ctx = new GenericXmlApplicationContext(path);
        
        UserDao userDao = ctx.getBean("userDao", UserDao.class);
        User user = new User();
        user.setId("id");
        user.setPw("pw");
        user.setName("name");
        
        userDao.insert(user);
        
        User testUser = userDao.select("id");
        assertThat(testUser.getPw(), is(user.getPw()));
        assertThat(testUser.getName(), is(user.getName())); 
    }
    
    public static void main(String[] args) {
        JUnitCore.main("test.test.test.UserDaoTest");
    }
}
```

이렇게 @Test를 이용한 테스트 코드를 작성하고
main() 메소드에서 JUnitCore 클래스의 정적 메소드인 main() 메소드를 이용해서
원하는 @Test가 붙은 테스트 코드를 실행시킬 수 있다.

그리고 실행한 @Test 코드 중에서 문제가 있다면 어느 테스트 코드에 어떤 문제가 생겼는지 출력해주고
몇 초만에 테스트를 실행하였는지도 출력해준다.

위에서는 main() 메소드에서 어떤 테스트 코드를 실행할지를 설정하였는데
만약 Eclipse(이클립스) IDE를 사용하고 있다면 JUnit을 이용한 Test 코드를 모두 실행할 수 있도록
지원하니 main() 메소드를 굳이 만들 필요는 없다.

#### 3-3. expected 요소 사용하기

위에서는 @Test를 사용할 때 아무 요소가 없는 어노테이션을 사용했다.
하지만 expected 속성을 사용하면 어떤 예외가 발생할 경우에만 테스트가 성공하는
테스트 코드를 작성할 수 있다.

만약 다음과 같이 사용한다면 NullPointerException이 발생할 경우에만 테스트 성공으로 간주한다.

```java
@Test(expected=NullPointerException.class)
```

### 4. @Test와 스프링의 관계

우리가 스프링을 이용하여 개발을 할 때 굉장히 간과하면서 개발하는 부분이 이 @Test 부분이다.
애초에 @Test를 사용하지 않는다는 것 자체가 스프링의 절반을 사용하지 않겠다는 것과 같은 말이기도 하고
스프링을 개발하면서 @Test 어노테이션을 사용하긴 하는데 가장 치명적인 문제가
성공하는 코드만을 작성하여 확인한다는 점이다.

만약 오류가 있는 코드인데도 불구하고 오류가 발생하지 않는 테스트만 하여 오류가 있는지 알지 못한 상태로
서비스를 시작하기라도 한다면 정말 큰 문제가 발생할 것이다.

그래서 나온 개발 방법이 먼저 테스트 코드를 만들고 기능을 구현하는 개발 이론이다.
테스트 코드를 먼저 만들고 기능을 구현하게 되면
테스트 코드를 만들어서 실행했을 때 무조건 실패하게 된다.
이 점이 이상하다고 느껴질 수도 있다.
무조건 실패하는데 그 뒤에 기능을 구현한다는 점이 말이다.

하지만 이렇게 테스트 코드를 만든 뒤에 기능을 구현하게 되면
다방면에서 생각한 테스트를 테스트 코드를 통해 만들게 되고 그 테스트 코드가 만족하는 기능을 만들어야
오류가 발생하지 않기 때문에
테스트 코드를 만족하는 기능을 구현했을 때 우리는 기능을 구현함과 동시에 테스트도 모두 거쳤다고 볼 수 있다.
그래서 이러한 테스트 코드를 먼저 작성하고 기능을 후에 개발하는 개발 방법을 많이 사용한다.