## @ComponentScan

@ComponentScan은 @Component를 포함하여 그로 파생된 @Service, @Repository, @Controller를
스프링 설정 파일에서 인식할 수 있도록 스캔하는 역할을 한다.
기존에 XML로 구성된 스프링 설정 파일에서는 context:component:scan이라는 태그를 이용해서
@Component 종류의 어노테이션이 붙은 클래스 파일을 찾아냅니다.
그렇게 기존에 XML에서는 context:component:scan을 이용해서 파일 찾기를 했지만
자바 코드 즉, @Configuration이 붙은 클래스에서는 @ComponentScan를 이용해서
파일 찾기를 시도할 수 있다.

ComponentScan의 요소는 다음과 같다.

```java
basePackageClasses
basePackages
excludeFilters
includeFilters
lazyInit
nameGenerator
resourcePattern
scopedProxy
scopeResolver
useDefaultFilters
value
```

이렇게 굉장히 많은 요소들을 가지고 있는데
하나 하나 한 번 살펴보도록 하겠다.

### 1. basePackages

basePackages 요소는 어느 패키지에서 어노테이션을 찾을 것인지 기본적인 베이스를 알려주는 요소이다.
만약 abc.def.ghi 패키지안의 클래스들을 확인하고 싶다면
다음과 같이 어노테이션을 사용할 수 있다.

```java
@ComponentScan(basePackages = "abc.def.ghi")
```

basePackages 요소의 값은 String[] 타입이기 때문에 여러 패키지들을 스캔하고 싶다면
배열 형식으로 작성해도 된다.
만약 aaa, bbb, ccc 패키지를 스캔하고 싶다면 다음과 같이 basePackages 요소를 사용할 수 있다.

```java
@ComponentScan(basePackages = {"aaa", "bbb", "ccc"})
```

### 2. basePackageClasses

basePackageClasses는 전에 basePackages를 배웠으니 이름만 봐도 대충 유추할 수 있듯이
패키지가 아니라 더 자세하게 클래스를 지정하는 것이다.
하지만 패키지를 지정하지 않으므로 같은 패키지 내에 있는 클래스들을 작성해야 한다.
그리고 타입이 Class<?>[] 타입이므로 Class 하나만 사용할 수도 있고 배열 형태로 사용할 수도 있다.
따라서 TestClass 클래스를 스캔하고 싶다면 다음과 같이 basePackageClasses 요소를 사용할 수 있다.

```java
@ComponentScan(basePackageClasses = TestClass.class)
```

이럴 땐 반드시 TestClass 클래스가 이 어노테이션을 붙인 클래스와 같은 패키지에 존재해야 한다.
그리고 스캔할 클래스가 여러 개라면 다음과 같이 작성할 수 있다.

```java
@ComponentScan(basePackageClasses = {TestClass1.class,
                                     TestClass2.class,
                                     TestClass3.class})
```

### 3. useDefaultFilters

useDefaultFilters 요소는 디폴트 값으로 스캔을 할 것인지를 선택한다.
기본적으로 @Component, @Repository, @Service, @Controller가 붙은 클래스는
자동으로 스캔하여 빈으로 만들어준다.
그것은 useDefaultFilters 요소의 디폴트 값이 true이기 때문이다.
만약 이후에 설명할 includeFilters 요소를 사용하기 위해서는
반드시 useDefaultFilters 요소의 값을 false로 해야합니다.

useDefaultFilters 요소는 위에서 말했던 것처럼 boolean 타입의 값을 가지기 때문에
다음과 같이 사용할 수 있습니다.

```java
@ComponentScan(useDefaultFilters = false)
```

### 4. includeFilters

includeFilters 요소는 @Filter를 받는데 이 @Filter를 이용해서 스캔을 할 수 있도록 지원한다.
@Filter는 어떤 방식으로 스캔할 클래스를 얻는지 알려준다.
@Filter의 요소로는 type과 그 type에 맞는 요소 하나가 있다.
type에는 FilterType 타입의 요소가 들어갈 수 있는데
ENUM으로 정의되어 있어서 쉽게 사용할 수 있다.
ENUM으로 정의되어 있는 요소는 다음과 같다.

```java
FilterType.REGEX;
FilterType.ANNOTATION;
FilterType.ASPECTJ;
FilterType.ASSIGNABLE_TYPE;
FilterType.CUSTOM;
```

이 type을 어떻게 정하느냐에 따라서 어떻게 스캔할 클래스들을 얻는지가 달라진다.

#### 4-1. FilterType.ANNOTATION

FilterType.ANNOTATION은 어느 어노테이션이 작성된 클래스인가에 따라서 스캔을 하도록 명시하는
FilterType이다.
@Filter의 두 번째 요소로 classes를 사용할 수 있다.
classes 요소에서는 Class<?>[] 타입을 받는다.
그리고 그 타입에는 어노테이션의 타입을 작성한다.
만약 @Repository, @Controller가 작성된 클래스만 스캔하고 싶다면 다음과 같이 작성하면 된다.

```java
@ComponentScan(
	includeFilters = {
        @Filter(
        	type = FilterType.ANNOTATION,
            classes = {Repository.class, Controller.class}
        )
    }
)
```

#### 4-2. FilterType.ASSIGNABLE_TYPE

FilterType.ASSIGNABLE_TYPE은 클래스를 기준으로 스캔을 하는 방법을 제공한다.
FilterType.ANNOTATION과 마찬가지로 classes 요소를 사용하기 때문에 클래스를 배열로 받을 수 있다.
만약 TestClass1, TestClass2, TestClass3 클래스를 스캔하고 싶다면 다음과 같이 작성하면 된다.

```java
@ComponentScan(
	includeFilters = {
        @Filter(
        	type = FilterType.ASSIGNABLE_TYPE,
            classes = {TestClass1.class, TestClass2.class, TestClass3.class}
        )
    }
)
```

#### 4-3. FilterType.REGEX

FilterType.REGEX는 정규 표현식을 이용해서 대상되는 이름의 패키지나 클래스를 스캔한다.
정규 표현식을 모른다면 찾아보길 바란다.
그리고 FilterType.REGEX에서는 이전의 FilterType과는 다르게 특정 클래스, 어노테이션을 스캔하는 것이 아니라,
어떠한 패턴을 이용하기 때문에 classes 요소가 아니라 pattern 요소를 사용한다.
pattern은 String[] 타입을 받는다.

만약 DAO가 붙은 패키지 또는 클래스들을 모두 스캔하고 싶다면 다음과 같이 사용할 수 있다.

```java
@ComponentScan(
	includeFilters = {
        @Filter(
			type = FilterType.REGEX,
            pattern = {"*DAO*"}
        )
    }
)
```

#### 4-4. FilterType.ASPECTJ

FilterType.ASPECTJ는 AspectJ 패턴을 이용하여 클래스 및 패키지를 스캔한다.
FilterType.REGEX와 마찬가지로 패턴을 이용해서 스캔하므로 classes 대신 pattern을 사용한다.
AspectJ 패턴은 메소드의 형식을 이용해서 스캔하는 방식을 사용한다.

#### 4-5. FilterType.CUSTOM

FilterType.CUSTOM은 이름에서 유추할 수 있듯이 개발자가 직접 정의하여 사용하는
어노테이션이라는 뜻이다.
직접 FilterType 클래스를 상속받아서 구현하면 직접 사용할 수 있다.

### 5. excludeFilters

excludeFilters는 includeFilter와는 완전히 반대의 의미이다.
excludeFilters는 기본적으로 모든 패키지를 스캔한다는 기준에서
어떠한 패키지 및 클래스를 뺀다는 뜻이다.
따라서 includeFilters와 사용하는 방법이 같다.

### 6. lazyInit

@ComponentScan을 이용해서 어노테이션들을 스캔하여 빈으로 등록할 때는
스캔과 동시에 바로 빈으로 등록하게 된다.
하지만 lazyInit 요소를 true로 하면 빈으로 바로 등록하지 않고
사용될 때 바로 빈으로 등록된다.
사용하는 방법은 다음과 같다.

```java
@ComponentScan(lazyInit = true)
```

### 7. resourcePattern

resourcePattern 요소는 String 타입을 하나 받으며,
다른 요소와는 다르게 배열로 받는 게 아니라 딱 하나만 받기 때문에
이것만을 명시하고 싶을 때 사용합니다.
그리고 정규 표현식을 사용할 수 있기 때문에
다음과 같이 사용하면 같은 패키지에 존재하는 모든 클래스를 스캔합니다.

```java
@ComponentScan(resourcePattern = "*.class")
```

### 8. scopedProxy

scopedProxy는 설정한 스코프 내에서 프록시 객체를 생성할 것인지에 대한 여부를 묻는 어노테이션이다.
받는 값은 ScopedProxyMode이고 ScopedProxyMode 객체는 ENUM이기 때문에
쉽게 사용할 수 있다. ScopedProxyMode의 필드는 다음과 같다.

```java
ScopedProxyMode.DEFAULT;
ScopedProxyMode.INTERFACES;
ScopedProxyMode.NO;
ScopedProxyMode.TARGET_CLASS;
```

#### 8-1. ScopedProxyMode.DEFAULT

ScopedProxyMode.DEFAULT는 scopedProxy 요소를 설정하지 않았을 경우
디폴트로 설정되는 옵션이다.
이름도 디폴트로 되어 있는데 기본적으로 ScopedProxyMode.NO와 같은 역할을 한다.

#### 8-2. ScopedProxyMode.INTERFACES

ScopedProxyMode.INTERFACES는 프록시 객체를 생성할 수 있는 구문이 존재한다면
프록시 객체를 생성하라는 속성이다.

#### 8-3. ScopedProxyMode.NO

ScopedProxyMode.NO는 프록시 객체를 생성하지 않도록 설정한다.

#### 8-4. ScopedProxyMode.TARGET_CLASS

ScopedProxyMode.TARGET_CLASS는 CGLIB 라이브러리를 사용해서 프록시를 생성한다.
CGLIB는 Code Generator LIBrary의 약자로서 동적으로 프록시 객체를 생성할 수 있도록
도와주는 라이브러리이다.

이렇게 scopedProxy 요소를 사용하게 되면 scopeResolver라는 요소의 값은 무시된다는 점을 기억하자.

### 9. scopeResolver

scopeResolver 요소는 ScopeMetadataResolver 타입 또는 이를 상속받은 클래스를 받는다.
이 ScopeMetadataResolver를 이용해서 스캔한 구성 요소의 범위를 분석한다.

### 10. nameGenerator

nameGenerator 요소는 BeanNameGenerator 객체를 받아서 사용하는데
이 BeanNameGenerator 객체를 이용하면 자동으로 AnnotationBeanNameGenerator 클래스 또는
스프링 부트라면 기본적으로 제공하는 ApplicationContext에 제공된 인스턴스를 이용해서
스프링 컨테이너 내에서 스캔된 컴포넌트의 이름을 지정하는데 사용된다.

### 11. value

value 요소는 basePackages의 별명으로 사용된다.
기본적으로 어노테이션에 요소의 이름을 작성하지 않고 그냥 값만 입력하게 되면 value 요소가 작성되는데
요소를 작성하지 않으면 basePackages가 작동하도록 별명을 부여한 것이다.