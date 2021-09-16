# 빈 스코프

1. 빈 스코프란?
2. 프로토타입 스코프
3. 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 문제점
4. 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 Provider로 문제 해결
5. 웹 스코프
6. request 스코프 예제 만들기
7. 스코프와 Provider
8. 스코프와 프록시

<br>

## 1. 빈 스코프란?

스프링 빈이 스프링 컨테이너의 시작과 함께 생성되어서 스프링 컨테이너가 종료될 때 까지 유지된다고 배움
-> 스프링 빈이 기본적으로 싱글톤 스코프로 생성되기 때문

<br>

### 스코프

- 빈이 존재할 수 있는 범위
  **스프링은 다양한 스코프를 지원**
- 싱글톤: 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
- 프로토타입: 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입 + 초기화 메서드까지만 불러주고 더는 관리하지 않는 매우 짧은 범위의 스코프
- 웹 관련 스코프
  - request: 웹 요청이 들어오고 나갈때 까지 유지되는 스코프
  - session: 웹 세션이 생성되고 종료될 때 까지 유지
  - application: 웹의 서블릿 컨텍스트와 같은 범위로 유지(굉장히 긴 범위)

빈 스코프는 다음과 같이 지정할 수 있다.

- 컴포넌트 스캔 자동 등록

```java
Scope("prototype")
@Component
public class HelloBean {}
```

- 수동 등록

```java
@Scope("prototype")
@Bean
PrototypeBean HelloBean() {
    return new HelloBean();
}
```

<br>

## 2. 프로토타입 스코프

- 싱글톤 스코프의 빈을 조회
  - 스프링 컨테이너는 항상 같은 인스턴스의 스프링 빈을 반환
- 프로토타입 스코프를 스프링 컨테이너에 조회
  - 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환

<br>

### 싱글톤 빈 요청

1. 싱글톤 스코프의 빈을 스프링 컨테이너에 요청
2. 스프링 컨테이너는 본인이 관리하는 스프링 빈을 반환
3. 같은 요청이 와도 같은 객체 인스턴스의 스프링 빈을 반환(공유해서 사용)

<br>

### 프로토타입 빈 요청

1. 프로토타입 스코프의 빈을 스프링 컨테이너에 요청
2. 스프링 컨테이너는 이 시점에 프로토타입 빈을 생성하고, 필요한 의존관계를 주입
3. 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환하고 끝(버림)
4. 이후에 스프링 컨테이너에 같은 요청이 오면 항상 새로운 프로토타입 빈을 생성해서 반환

<br>

### 정리

- 스프링 컨테이너는 프로토타입 빈을 생성하고, 의존관계 주입, 초기화까지만 처리한다는 것
- 클라이언트에 빈을 반환하고, 이후 스프링 컨테이너는 생성된 프로토타입 빈을 관리하지 않음
- 프로토타입 빈을 관리할 책임은 프로토타입 빈을 받은 클라이언트에게 있음.
- 그래서 @PreDestory 같은 메서드가 호출되지 않는다.

<br>

### 싱글톤 스코프 빈 조회 테스트

```java
public class SingletonTest {

    @Test
    public void singletonBeanFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SingletonBean.class);

        SingletonBean singletonBean1 = ac.getBean(SingletonBean.class);
        SingletonBean singletonBean2 = ac.getBean(SingletonBean.class);
        System.out.println("singletonBean1 = " + singletonBean1);
        System.out.println("singletonBean2 = " + singletonBean2);

        assertThat(singletonBean1).isSameAs(singletonBean2);
        ac.close(); //종료
    }

    @Scope("singleton")
    static class SingletonBean {

        @PostConstruct
        public void init() {
            System.out.println("SingletonBean.init");
        }

        @PreDestroy
        public void destroy() {
            System.out.println("SingletonBean.destroy");
        }
        }
}
```

<br>

### 실행 결과

```bash
SingletonBean.init
singletonBean1 = hello.core.scope.PrototypeTest$SingletonBean@54504ecd
singletonBean2 = hello.core.scope.PrototypeTest$SingletonBean@54504ecd
org.springframework.context.annotation.AnnotationConfigApplicationContext -
Closing SingletonBean.destroy
```

- 빈 초기화 메서드를 실행하고, 같은 인스턴스의 빈을 조회하고, 종료 메서드까지 정상 호출 된 것을 확인

<br>

### 프로토타입 스코프 빈 조회 테스트

```java
public class PrototypeTest {
    @Test
    public void prototypeBeanFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);

        System.out.println("find prototypeBean1");
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        System.out.println("find prototypeBean2");
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);

        System.out.println("prototypeBean1 = " + prototypeBean1);
        System.out.println("prototypeBean2 = " + prototypeBean2);

        assertThat(prototypeBean1).isNotSameAs(prototypeBean2);
        ac.close(); //종료
    }

    @Scope("prototype")
    static class PrototypeBean {

        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init");
        }

        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```

<br>

### 실행 결과

```bash
find prototypeBean1
PrototypeBean.init
find prototypeBean2
PrototypeBean.init
prototypeBean1 = hello.core.scope.PrototypeTest$PrototypeBean@13d4992d
prototypeBean2 = hello.core.scope.PrototypeTest$PrototypeBean@302f7971
org.springframework.context.annotation.AnnotationConfigApplicationContext -
Closing
```

<br>

### 싱글톤과 프로토타입의 차이점

- 싱글톤 빈
  - 스프링 컨테이너 생성 시점에 초기화 메서드가 실행
  - 스프링 컨테이너가 관리하기 때문에 스프링 컨테이너가 종료될 때 빈의 종료 메서드가 실행
- 프로토타입 스코프의 빈
  - 스프링 컨테이너에서 빈을 조회할 때 생성되고, 초기화 메서드도 실행
  - 2번 조회했으므로 완전히 다른 스프링 빈이 생성되고, 초기화도 2번 실행됨
  - 스프링 컨테이너가 생성과 의존관계 주입 그리고 초기화 까지만 관여하고, 더는 관리하지 않음
    -> 스프링 컨테이너가 종료될 때 @PreDestroy 같은 종료 메서드가 전혀 실행되지 않음

<br>

### 프로토타입 빈의 특징 정리

- 스프링 컨테이너에 요청할 때 마다 새로 생성
- 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입 그리고 초기화까지만 관여한다. 종료 메서드가 호출되지 않음
  -> 프로토타입 빈은 프로토타입 빈을 조회한 클라이언트가 관리해야 함
  -> 종료 메서드에 대한 호출도 클라이언트가 직접 해야 함

<br>

## 3. 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 문제점

- 스프링 컨테이너에 프토토타입 스코프의 빈을 요청하면 항상 새로운 객체 인스턴스를 생성해서 반환
- 하지만 싱글톤 빈과 함께 사용할 때는 의도한 대로 잘 동작하지 않으므로 주의!!

<br>

### 프로토타입 빈 직접 요청

- 스프링 컨테이너에 프로토타입 빈 직접 요청1
  - 클라이언트A는 스프링 컨테이너에 프로토타입 빈을 요청
  - 스프링 컨테이너는 프로토타입 빈을 새로 생성해서 반환(x01)
    > 해당 빈의 count 필드 값은 0이다.
  - 클라이언트는 조회한 프로토타입 빈에 count 필드를 +1하는 addCount() 를 호출
  - 프로토타입 빈(x01)의 count는 1
- 스프링 컨테이너에 프로토타입 빈 직접 요청2
  - 클라이언트B는 스프링 컨테이너에 프로토타입 빈을 요청
  - 스프링 컨테이너는 프로토타입 빈을 새로 생성해서 반환(x02)
    > 해당 빈의 count 필드 값은 0이다.
  - 클라이언트는 조회한 프로토타입 빈에 count 필드를 +1하는 addCount() 를 호출
  - 프로토타입 빈(x02)의 count는 1

<br>

#### 코드

```java
assertThat(prototypeBean2.getCount()).isEqualTo(1);


@Scope("prototype")
static class PrototypeBean {


private int count = 0;


public void addCount() { count++;
}



public int getCount() { return count;
}



@PostConstruct public void init() {
System.out.println("PrototypeBean.init " + this);

}



@PreDestroy

public void destroy() { System.out.println("PrototypeBean.destroy");
}

}


}

```

### 싱글톤 빈에서 프로토타입 빈 사용

clientBean 이라는 싱글톤 빈이 의존관계 주입을 통해서 프로토타입 빈을 주입받아서 사용하는 예

- 싱글톤에서 프로토타입 빈 사용1

  - clientBean 은 싱글톤이므로, 보통 스프링 컨테이너 생성 시점에 함께 생성되고, 의존관계 주입도 발생(자동 주입)
  - 주입 시점에 스프링 컨테이너에 프로토타입 빈을 요청
  - 스프링 컨테이너는 프로토타입 빈을 생성해서 clientBean 에 반환. 프로토타입 빈의 count 필드 값은 0이다.
  - clientBean은 프로토타입 빈을 내부 필드에 보관(정확히는 참조값을 보관한다.)

- 싱글톤에서 프로토타입 빈 사용2

  - clientBean 이 반환
  - 클라이언트 A는 clientBean.logic() 을 호출
  - clientBean 은 prototypeBean의 addCount() 를 호출. 프로토타입 빈의 count를 증가
    > count값이 1이 된다.

- 싱글톤에서 프로토타입 빈 사용3
  - clientBean 이 반환된다.
  - clientBean이 **내부에 가지고 있는 프로토타입 빈**은 이미 **과거에 주입이 끝난 빈**이다. 주입 시점에 스프링 컨테이너에 요청해서 프로토타입 빈이 새로 생성이 된 것이지, 사용 할 때마다 새로 생성되는 것이 아님
  - 클라이언트 B는 clientBean.logic() 을 호출한
  - clientBean 은 prototypeBean의 addCount() 를 호출해서 프로토타입 빈의 count를 증가
    > 원래 count 값이 1이었으므로 2가 됨

### 테스트 코드

```java
public class SingletonWithPrototypeTest1 {



@Test

void singletonClientUsePrototype() { AnnotationConfigApplicationContext ac = new
AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);


ClientBean clientBean1 = ac.getBean(ClientBean.class); int count1 = clientBean1.logic(); assertThat(count1).isEqualTo(1);

ClientBean clientBean2 = ac.getBean(ClientBean.class); int count2 = clientBean2.logic(); assertThat(count2).isEqualTo(2);
}


static class ClientBean {



private final PrototypeBean prototypeBean;



@Autowired

public ClientBean(PrototypeBean prototypeBean) { this.prototypeBean = prototypeBean;
}



public int logic() { prototypeBean.addCount();
int count = prototypeBean.getCount(); return count;
}

}


@Scope("prototype")
 static class PrototypeBean {
 private int count = 0;
 public void addCount() {
 count++;
 }
 public int getCount() {
 return count;
 }
 @PostConstruct
 public void init() {
 System.out.println("PrototypeBean.init " + this);
 }
 @PreDestroy
 public void destroy() {
 System.out.println("PrototypeBean.destroy");
 }
 }
}
```

### 정리

스프링은 일반적으로 싱글톤 빈을 사용 -> 싱글톤 빈이 프로토타입 빈을 사용하게 됨
그런데 싱글톤 빈은 생성 시점에만 의존관계 주입을 받기 때문에, 프로토타입 빈이 **새로 생성되기는 하지만, 싱글톤 빈과 함께 계속 유지**되는 것이 문제

- 프로토타입 빈을 주입 시점에만 새로 생성하는게 아니라, 사용할 때 마다 새로 생성해서 사용하는 것을 원함

> 참고: 여러 빈에서 같은 프로토타입 빈을 주입 받으면, 주입 받는 시점에 각각 새로운 프로토타입 빈이 생성됨
> 예를 들어서 clientA, clientB가 각각 의존관계 주입을 받으면 각각 다른 인스턴스의 프로토타입 빈을 주입 받음
> clientA prototypeBean@x01
> clientB prototypeBean@x02
> 물론 사용할 때 마다 새로 생성되는 것은 아님

## 4. 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 Provider로 문제 해결

싱글톤 빈과 프로토타입 빈을 함께 사용 시, 어떻게 하면 사용할 때 마다 항상 새로운 프로토타입 빈을 생성할 수 있을까?

스프링 컨테이너에 요청

-> 싱글톤 빈이 프로토타입을 사용할 때 마다 스프링 컨테이너에 새로 요청하는 것

핵심 코드

```java
@Autowired
private ApplicationContext ac;
public int logic() {
// 로직 호출할 때마다 컨테이너에서 받기
 PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
 prototypeBean.addCount();
 int count = prototypeBean.getCount();
 return count;
}
```

- 실행해보면 ac.getBean() 을 통해서 항상 새로운 프로토타입 빈이 생성되는 것을 확인가능
- 의존관계를 외부에서 주입(DI) 받는게 아니라 이렇게 **직접 필요한 의존관계를 찾는 것**을 **Dependency Lookup (DL) 의존관계 조회(탐색)**
- 그런데 이렇게 스프링의 애플리케이션 컨텍스트 전체를 주입받게 되면, 스프링 컨테이너에 종속적인 코드가 되고, 단위 테스트도 어려워짐
- 필요한 기능은 지정한 프로토타입 빈을 컨테이너에서 대신 찾아주는 딱! DL 정도의 기능만 제공하는 무언가가 필요

### ObjectFactory, ObjectProvider

- 지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공하는 것
  > 참고로 과거에는 ObjectFactory 가 있었는데, 여기에 편의 기능을 추가한 것

```java
@Autowired
private ObjectProvider<PrototypeBean> prototypeBeanProvider;
public int logic() {
 PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
 prototypeBean.addCount();
 int count = prototypeBean.getCount();
 return count;
}
```

- prototypeBeanProvider.getObject() 을 통해서 항상 새로운 프로토타입 빈이 생성됨을 확인
- ObjectProvider 의 getObject() 를 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환 (DL)
- 스프링이 제공하는 기능을 사용하지만, 기능이 단순하므로 단위테스트를 만들거나 mock 코드를 만들기는 훨씬 쉬워짐
- ObjectProvider 는 지금 딱 필요한 DL 정도의 기능만 제공

### 특징

- ObjectFactory
  - 기능이 단순, 별도의 라이브러리 필요 없음, 스프링에 의존
- ObjectProvider
  - ObjectFactory 상속, 옵션, 스트림 처리등 편의 기능이 많고, 별도의 라이브러리 필요 없음, 스프링에 의존

### 다른 방법(JSR-330 Provider)

- javax.inject.Provider 라는 JSR-330 자바 표준을 사용하는 방법
- 사용하려면 javax.inject:javax.inject:1 라이브러리를 gradle에 추가

javax.inject.Provider 참고용 코드

```java
package javax.inject;
public interface Provider<T> {
 T get();
}
//implementation 'javax.inject:javax.inject:1' gradle 추가 필수
@Autowired
private Provider<PrototypeBean> provider;
public int logic() {
 PrototypeBean prototypeBean = provider.get();
 prototypeBean.addCount();
 int count = prototypeBean.getCount();
 return count;

```

- provider.get() 을 통해서 항상 새로운 프로토타입 빈이 생성되는 것을 확인
- provider 의 get() 을 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환 (DL)
- 자바 표준이고, 기능이 단순하므로 단위테스트를 만들거나 mock 코드를 만들기는 훨씬 쉬워짐
- Provider 는 지금 딱 필요한 DL 정도의 기능만 제공

### 특징

메서드 하나로 기능이 매우 단순하다. 별도의 라이브러리가 필요하다.
자바 표준이므로 스프링이 아닌 다른 컨테이너에서도 사용할 수 있다.

### 정리

프로토타입 빈은 언제 사용할까?

- 매번 사용할 때 마다 의존관계 주입이 완료된 새로운 객체가 필요하면 사용
- 그런데 실무에서 웹 애플리케이션을 개발해보면, 싱글톤 빈으로 대부분의 문제를 해결할 수 있기 때문에 프로토타입 빈을 직접적으로 사용하는 일은 매우 드뭄
- ObjectProvider , JSR330 Provider 등은 프로토타입 뿐만 아니라 DL이 필요한 경우는 언제든지 사용 가능

> 참고: 스프링이 제공하는 메서드에 @Lookup 애노테이션을 사용하는 방법도 있지만, 이전 방법들로 충분하고, 고려해야할 내용도 많아서 생략

> 참고: 실무에서 자바 표준인 JSR-330 Provider를 사용할 것인지, 아니면 스프링이 제공하는 ObjectProvider를 사용할 것인지 고민이 될 것이다. ObjectProvider는 DL을 위한 편의 기능을 많이 제공해주고 스프링 외에 별도의 의존관계 추가가 필요 없기 때문에 편리하다. 만약(정말 그럴일은 거의 없겠지만) 코드를 스프링이 아닌 다른 컨테이너에서도 사용할 수 있어야 한다면 JSR-330 Provider를 사용해야함
>
> 스프링을 사용하다 보면 이 기능 뿐만 아니라 다른 기능들도 자바 표준과 스프링이 제공하는 기능이 겹칠때가 많이 있다. 대부분 스프링이 더 다양하고 편리한 기능을 제공해주기 때문에, 특별히 다른 컨테이너를 사용할 일이 없다면, 스프링이 제공하는 기능을 사용하면 됨

## 5. 웹 스코프

### 웹 스코프의 특징

- 웹 환경에서만 동작한다.
- 스프링이 해당 스코프의 종료시점까지 관리
  > 따라서 종료 메서드가 호출됨

### 웹 스코프 종류

#### request

- HTTP 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프
- 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리

#### session

- HTTP Session과 동일한 생명주기를 가지는 스코프

#### application

- 서블릿 컨텍스트( ServletContext )와 동일한 생명주기를 가지는 스코프

#### websocket

- 웹 소켓과 동일한 생명주기를 가지는 스코프

## 6. request 스코프 예제 만들기

#### 웹 환경 추가

웹 스코프는 웹 환경에서만 동작하므로 web 환경이 동작하도록 라이브러리를 추가

#### build.gradle에 추가

```java
//web 라이브러리 추가
implementation 'org.springframework.boot:spring-boot-starter-web'
```

이제 hello.core.CoreApplication 의 main 메서드를 실행하면 웹 애플리케이션이 실행되는 것을 확인할 수 있다.

```bash
Tomcat started on port(s): 8080 (http) with context path ''
Started CoreApplication in 0.914 seconds (JVM running for 1.528)
```

> 참고: spring-boot-starter-web 라이브러리를 추가하면 스프링 부트는 내장 톰켓 서버를 활용해서 웹 서버와 스프링을 함께 실행시킨다.

> 참고: 스프링 부트는 웹 라이브러리가 없으면 우리가 지금까지 학습한
> AnnotationConfigApplicationContext 을 기반으로 애플리케이션을 구동한다. 웹 라이브러리가 추가되면 웹과 관련된 추가 설정과 환경들이 필요하므로
> AnnotationConfigServletWebServerApplicationContext 를 기반으로 애플리케이션을 구동한다.

#### 포트로 변경 설정을 추가

```java
// main/resources/application.properties
server.port=9090
```

- 기본 포트 8080 시 오류를 잡아준다

### request 스코프 예제 개발

- 동시에 여러 HTTP 요청이 오면 정확히 어떤 요청이 남긴 로그인지 구분하기 어려움  
  -> 이럴때 사용하기 딱 좋은것이 바로 request 스코프.

- 예시 로그 형식처럼 남도록 request 스코프를 활용해서 추가 기능을 개발해보자.

#### 예시 로그

```bash
[d06b992f...] request scope bean create
[d06b992f...][http://localhost:8080/log-demo] controller test
[d06b992f...][http://localhost:8080/log-demo] service id = testId
[d06b992f...] request scope bean clos
```

- 기대하는 공통 포멧: [UUID][requesturl] {message}
- UUID를 사용해서 HTTP 요청을 구분하자.(전 세계 유일한 ID, 같으면 동일 고객)
- requestURL 정보도 추가로 넣어서 어떤 URL을 요청해서 남은 로그인지 확인하자.

#### MyLogger

```java
package hello.core.common;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import java.util.UUID;
@Component
@Scope(value = "request")
public class MyLogger {
 private String uuid;
 private String requestURL;
 public void setRequestURL(String requestURL) {
 this.requestURL = requestURL;
 }
 public void log(String message) {
 System.out.println("[" + uuid + "]" + "[" + requestURL + "] " +
message);
 }
 @PostConstruct // 생성 시점 자동 초기화 메서드, uuid 생성 & 저장
 public void init() {
 uuid = UUID.randomUUID().toString();
 System.out.println("[" + uuid + "] request scope bean create:" + this);
 }
 @PreDestroy // 종료 메시지 남김
 public void close() {
 System.out.println("[" + uuid + "] request scope bean close:" + this);
 }
}
```

- 로그를 출력하기 위한 클래스
- @Scope(value = "request") 를 사용해서 request 스코프로 지정
  - 이제 이 빈은 HTTP 요청 당 하나씩 생성되고, HTTP 요청이 끝나는 시점에 소멸됨
- 이 빈이 생성되는 시점에 자동으로 @PostConstruct 초기화 메서드를 사용해서 uuid를 생성해서 저장
- 이 빈은 HTTP 요청 당 하나씩 생성되므로, uuid를 저장해두면 다른 HTTP 요청과 구분 가능
- 이 빈이 소멸되는 시점에 @PreDestroy 를 사용해서 종료 메시지를 남긴다.
- requestURL 은 이 빈이 생성되는 시점에는 알 수 없으므로, 외부에서 setter로 입력 받음

#### LogDemoController

```java
package hello.core.web;
import hello.core.common.MyLogger;
import hello.core.logdemo.LogDemoService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import javax.servlet.http.HttpServletRequest;
@Controller
@RequiredArgsConstructor
public class LogDemoController {
 private final LogDemoService logDemoService;
 private final MyLogger myLogger;
 @RequestMapping("log-demo")
 @ResponseBody
 public String logDemo(HttpServletRequest request) {
 String requestURL = request.getRequestURL().toString();
 myLogger.setRequestURL(requestURL);
 myLogger.log("controller test");
 logDemoService.logic("testId");
 return "OK";
 }
}
```

- 로거가 잘 작동하는지 확인하는 테스트용 컨트롤러
- HttpServletRequest를 통해서 요청 URL을 받음
  - requestURL 값 http://localhost:8080/log-demo
- requestURL 값을 myLogger에 저장
- myLogger는 HTTP 요청 당 각각 구분되므로 다른 HTTP 요청 때문에 값이 섞이는 걱정 안해도 됨
- 컨트롤러에서 controller test라는 로그를 남김

> 참고: requestURL을 MyLogger에 저장하는 부분은 컨트롤러 보다는 공통 처리가 가능한 스프링 인터셉터나 서블릿 필터 같은 곳을 활용하는 것이 좋다. 여기서는 예제를 단순화하고, 아직 스프링 인터셉터를 학습하지 않은 분들을 위해서 컨트롤러를 사용했다. 스프링 웹에 익숙하다면 인터셉터를 사용해서 구현해보자.

LogDemoService 추가

```java
package hello.core.logdemo;
import hello.core.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
@Service
@RequiredArgsConstructor
public class LogDemoService {
 private final MyLogger myLogger;
 public void logic(String id) {
 myLogger.log("service id = " + id);
 }
}
```

- 비즈니스 로직이 있는 서비스 계층에서도 로그를 출력
- request scope의 MyLogger 덕분에 이런 부분을 파라미터로 넘기지 않고, MyLogger의 멤버변수에 저장해서 코드와 계층을 깔끔하게 유지 가능
  > request scope를 사용하지 않고 파라미터로 이 모든 정보를 서비스 계층에 넘긴다면, 파라미터가 많아서 지저분짐 더 문제는 requestURL 같은 웹과 관련된 정보가 웹과 관련없는 서비스 계층까지 넘어가게 됨 웹과 관련된 부분은 컨트롤러까지만 사용해야 함 서비스 계층은 웹 기술에 종속되지 않고, 가급적 순수하게 유지하는 것이 유지보수 관점에서 좋음

#### 기대하는 출력

```bash
[d06b992f...] request scope bean create
[d06b992f...][http://localhost:8080/log-demo] controller test
[d06b992f...][http://localhost:8080/log-demo] service id = testId
[d06b992f...] request scope bean close
```

#### 실제 출력

```bash
Error creating bean with name 'myLogger': Scope 'request' is not active for the
current thread; consider defining a scoped proxy for this bean if you intend to
refer to it from a singleton;

```

- 실제는 기대와 다르게 애플리케이션 실행 시점에 오류 발생
- 스프링 애플리케이션을 실행하는 시점에 싱글톤 빈은 생성해서 주입이 가능하지만, request 스코프 빈은 아직 생성되지 않음
  - 이 빈은 실제 고객의 요청이 와야 생성 가능
    -> 스프링 컨테이너에게 나의 스프링 빈을 주세요라는 단계를 의존관계 주입 단계가 아니라 실제 고객 요청이 왔을 때로 지연시켜야 함

## 7. 스코프와 Provider

### 앞서 배운 Provider를 사용하는 것

간단히 ObjectProvider를 사용해보자.

```java
public class LogDemoController {
 private final LogDemoService logDemoService;
 private final ObjectProvider<MyLogger> myLoggerProvider;
 @RequestMapping("log-demo")
 @ResponseBody
 public String logDemo(HttpServletRequest request) {
 String requestURL = request.getRequestURL().toString();
 MyLogger myLogger = myLoggerProvider.getObject(); // provider 사용
 myLogger.setRequestURL(requestURL);
 myLogger.log("controller test");
 logDemoService.logic("testId");
 return "OK";
 }
}
```

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {
 private final ObjectProvider<MyLogger> myLoggerProvider; // provider 사용
 public void logic(String id) {
 MyLogger myLogger = myLoggerProvider.getObject();
 myLogger.log("service id = " + id);
 }
}
```

- 덕분에 ObjectProvider.getObject() 를 호출하는 시점(해당 주문이 불려지는 시점)까지 request scope 빈의 생성을 지연 가능
- ObjectProvider.getObject() 를 호출하시는 시점에는 HTTP 요청이 진행중이므로 request scope 빈의 생성이 정상 처리됨
- ObjectProvider.getObject() 를 LogDemoController , LogDemoService에서 각각 한번씩 따로 호출해도 같은 HTTP 요청이면 같은 스프링 빈이 반환
- 요청마다 객체를 따로 관리 가능하게 함

## 8. 스코프와 프록시

### 프록시 방식

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {
}
```

- MyLogger의 가짜 프록시 클래스를 만들어두고 HTTP request와 상관 없이 가짜 프록시 클래스를 다른 빈에 미리 주입 가능

#### 참고

> 여기가 핵심이다. proxyMode = ScopedProxyMode.TARGET_CLASS 를 추가해주자.
> 적용 대상이 인터페이스가 아닌 클래스면 TARGET_CLASS 를 선택 적용 대상이 인터페이스면 INTERFACES 를 선택

```java
package hello.core.web;
import hello.core.common.MyLogger;
import hello.core.logdemo.LogDemoService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import javax.servlet.http.HttpServletRequest;
@Controller
@RequiredArgsConstructor
public class LogDemoController {
 private final LogDemoService logDemoService;
 private final MyLogger myLogger;
 @RequestMapping("log-demo")
 @ResponseBody
 public String logDemo(HttpServletRequest request) {
 String requestURL = request.getRequestURL().toString();
 myLogger.setRequestURL(requestURL);
 myLogger.log("controller test");
 logDemoService.logic("testId");
 return "OK";
 }
}
```

```java
package hello.core.logdemo;
import hello.core.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
@Service
@RequiredArgsConstructor
public class LogDemoService {
 private final MyLogger myLogger;
 public void logic(String id) {
 myLogger.log("service id = " + id);
 }
}
```

- 실행하면 잘 동작된다
- 코드를 잘 보면 LogDemoController , LogDemoService 는 Provider 사용 전과 완전히 동일

### 웹 스코프와 프록시 동작 원리

#### myLogger 확인

```java
System.out.println("myLogger = " + myLogger.getClass());
```

#### 출력결과

```bash
myLogger = class hello.core.common.MyLogger$$EnhancerBySpringCGLIB$$b68b726d
```

- CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입
- @Scope 의 proxyMode = ScopedProxyMode.TARGET_CLASS) 를 설정하면 스프링 컨테이너는 CGLIB 라는 바이트코드를 조작하는 라이브러리를 사용해서, MyLogger를 상속받은 가짜 프록시 객체를 생성
- 이 프록시 객체가 provider의 역할을 수행
- 우리가 등록한 순수한 MyLogger 클래스가 아니라 MyLogger$$EnhancerBySpringCGLIB 이라는 클래스로 만들어진 객체가 대신 등록된 것을 확인(출력결과)
- 그리고 스프링 컨테이너에 "myLogger"라는 이름으로 진짜 대신에 이 가짜 프록시 객체를 등록
- ac.getBean("myLogger", MyLogger.class) 로 조회해도 프록시 객체가 조회되는 것을 확인할 수 있다.
  -> 그래서 의존관계 주입도 이 가짜 프록시 객체가 주입

- 가짜 프록시 객체는 **요청이 오면** 그때 내부에서 진짜 빈을 요청하는 위임 로직이 들어있음
- 가짜 프록시 객체는 내부에 진짜 myLogger를 찾는 방법을 알고 있음
- 클라이언트가 myLogger.logic() 을 호출하면 사실은 가짜 프록시 객체의 메서드를 호출한 것
- 가짜 프록시 객체는 request 스코프의 진짜 myLogger.logic() 를 호출
- 가짜 프록시 객체는 원본 클래스를 상속 받아서 만들어졌기 때문에 이 객체를 사용하는 클라이언트 입장에서는 사실 원본인지 아닌지도 모르게, 동일하게 사용 가능(다형성)

### 동작 정리

- CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입한다.
- 이 가짜 프록시 객체는 실제 요청이 오면 그때 내부에서 실제 빈을 요청하는 위임 로직이 들어있음
- 가짜 프록시 객체는 실제 request scope와는 관계가 없다. 그냥 가짜이고, 내부에 단순한 위임 로직만 있고, 싱글톤처럼 동작

### 특징 정리

- 프록시 객체 덕분에 클라이언트는 마치 싱글톤 빈을 사용하듯이 편리하게 request scope를 사용 가능
- 사실 Provider를 사용하든, 프록시를 사용하든 핵심 아이디어는 진짜 객체 조회를 꼭 필요한 시점까지 지연처리 한다는 점
- 단지 애노테이션 설정 변경만으로 원본 객체를 프록시 객체로 대체할 수 있다. 이것이 바로 다형성과 DI 컨테이너가 가진 큰 강점
- 꼭 웹 스코프가 아니어도 프록시는 사용 가능

### 주의점

- 마치 싱글톤을 사용하는 것 같지만 다르게 동작하기 때문에 결국 주의해서 사용해야 함
- 이런 특별한 scope는 꼭 필요한 곳에만 최소화해서 사용하자, 무분별하게 사용하면 유지보수하기 어려움
