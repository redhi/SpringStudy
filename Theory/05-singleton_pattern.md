# 싱글톤 컨테이너

1. 웹 애플리케이션과 싱글톤
2. 싱글톤 패턴
3. 싱글톤 컨테이너
4. 싱글톤 방식의 주의점
5. @Configuration과 싱글톤
6. @Configuration과 바이트코드 조작의 마법
<br>

## 1. 웹 애플리케이션과 싱글톤

스프링은 기업용 온라인 서비스 기술을 지원하기 위한 용도.

- 웹 애플리케이션은 보통 여러 고객이 동시에 요청을 한다.

<br>

### 스프링 없는 순수한 DI 컨테이너

- AppConfig는 요청을 할 때 마다 객체를 새로 생성한다.
  > 고객 트래픽이 초당 100이 나오면 초당 100개 객체가 생성되고 소멸
- 메모리 낭비가 심하다.
- 해결방안은 해당 객체가 딱 1개만 생성되고, 공유하도록 설계(싱글톤 패턴)

<br>

## 2. 싱글톤 패턴

- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.
  -> 객체 인스턴스를 2개 이상 생성하지 못하도록 막아야 한다.
- private 생성자를 사용(외부에서 임의로 new 키워드를 사용하지 못하도록 막음)

<br>

### 생성 예제

> main이 아닌 test 위치에 생성하자.

```java
package hello.core.singleton;

public class SingletonService {

    //1. static 영역에 객체를 딱 1개만 생성시켜 놈
    private static final SingletonService instance = new SingletonService();

    //2. public으로 열어서 객체 인스터스가 필요하면 이 static 메서드를 통해서만 조회하도록 허용
    public static SingletonService getInstance() {
        return instance;
}
```

1. static 영역에 객체 instance를 미리 하나 생성
2. 이 객체 인스턴스가 필요하면 오직 getInstance() 메서드를 통해서만 조회. 이 메서드를 호출하면 항상 같은 인스턴스를 반환
3. 딱 1개의 객체 인스턴스만 존재해야 하므로, 생성자를 private으로 막아서 혹시라도 외부에서 new 키워드로 객체 인스턴스가 생성되는 것을 막음.

<br>

### 테스트 코드

```java
@Test
@DisplayName("싱글톤 패턴을 적용한 객체 사용")
public void singletonServiceTest() {
    //private으로 생성자를 막아두었다. 컴파일 오류가 발생한다.
    //new SingletonService();

    //1. 조회: 호출할 때 마다 같은 객체를 반환
    SingletonService singletonService1 = SingletonService.getInstance();
    //2. 조회: 호출할 때 마다 같은 객체를 반환
    SingletonService singletonService2 = SingletonService.getInstance();

    //참조값이 같은 것을 확인
    System.out.println("singletonService1 = " + singletonService1); System.out.println("singletonService2 = " + singletonService2);

    // singletonService1 == singletonService2 assertThat(singletonService1).isSameAs(singletonService2);
}
```

- private으로 new 키워드를 막음
  > 호출할 때 마다 같은 객체 인스턴스를 반환한다
- 싱글톤 패턴을 구현하는 방법은 여러가지가 있다.
  > 여기서는 객체를 미리 생성해두는 가장 단순하고 안전한 방법을 선택했다.

<br>

### 싱글톤 패턴의 장점

- 싱글톤 패턴을 적용하면 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 

<br>

### 싱글톤 패턴 문제점

- 싱글톤 패턴을 구현하는 코드 자체가 많음
- 클라이언트가 구체 클래스에 의존
- DIP를 위반
- 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높음
- 내부 속성을 변경하거나 초기화 하기 어려움.
- private 생성자로 자식 클래스를 만들기 어려움
  > 유연성이 떨어진다.     
- 안티패턴으로 불리기도 한다.

<br>

## 3. 싱글톤 컨테이너

스프링 컨테이너는 싱글톤 패턴의 문제점을 해결 + 객체 인스턴스를 싱글톤(1개만 생성)으로 관리
-> 스프링 빈이 바로 싱글톤으로 관리되는 빈이다.

<br>

### 싱글톤 컨테이너

- 스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리
  - 컨테이너는 객체를 하나만 생성해서 관리
- 스프링 컨테이너는 싱글톤 컨테이너 역할
  - 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라 함
- 싱글턴 패턴의 모든 단점을 해결하면서 객체를 싱글톤으로 유지 가능
- DIP, OCP, 테스트, private 생성자로 부터 자유롭게 싱글톤을 사용 가능  

<br>

### 테스트 코드

```java
@Test
@DisplayName("스프링 컨테이너와 싱글톤")
void springContainer() {
  ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
  
  //1. 조회: 호출할 때 마다 같은 객체를 반환
  MemberService memberService1 = ac.getBean("memberService", MemberService.class);
  //2. 조회: 호출할 때 마다 같은 객체를 반환
  MemberService memberService2 = ac.getBean("memberService", MemberService.class);

  //참조값이 같은 것을 확인
  System.out.println("memberService1 = " + memberService1);
  System.out.println("memberService2 = " + memberService2);
  //memberService1 == memberService2

  assertThat(memberService1).isSameAs(memberService2);

}
```
<br>

### 싱글톤 컨테이너 적용 후

- 이미 만들어진 객체를 공유해서 효율적으로 재사용 가능
  > 참고: 스프링의 기본 빈 등록 방식은 싱글톤이지만, 싱글톤 방식만 지원하는 것은 아님. 요청할 때 마다 새로운 객체를 생성해서 반환하는 기능도 제공. 자세한 내용은 빈 스코프 참고

## 4. 싱글톤 방식의 주의점

싱글톤 패턴이든, 스프링 같은 싱글톤 컨테이너를 사용하든, 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지(stateful)하게 설계하면 안됨

> 무상태(stateless)로 설계!!

- 특정 클라이언트에 의존적인 필드가 있으면 안된다.
- 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다! 가급적 읽기만(Read-only)
- 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 함.
- 스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있다!!!
<br>

### 상태를 유지 경우 발생하는 문제점 예시 코드

```java
package hello.core.singleton;

public class StatefulService {

  private int price; //상태를 유지하는 필드

  public void order(String name, int price) {
    System.out.println("name = " + name + " price = " + price); 
    this.price = price; //여기가 문제 발생
  }

  public int getPrice() { 
    return price;
  }

}
```
<br>

### 상태를 유지 경우 발생하는 문제점 예시 코드

```java
package hello.core.singleton;

import org.assertj.core.api.Assertions; import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext; import
org.springframework.context.annotation.AnnotationConfigApplicationContext; import org.springframework.context.annotation.Bean;

public class StatefulServiceTest {

  @Test
  void statefulServiceSingleton() { 
    ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
    StatefulService statefulService1 = ac.getBean("statefulService", StatefulService.class);

    //TreadA: A사용자 10000원 주문
    statefulService1.order("userA",10000);
    // ThreadB: B사용자 20000원 주문
    statefulService2.order("userB", 20000);

    // ThreadA: 사용자A 주문 금액 조회(20000이 조회됨)
    int price = statefulService1.getPrice();
    System.out.println("price = "+price);

    Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
  }
  static class TestConfig {
    @Bean
    public StatefulService statefulService() {
      return new StatefulService();
    }
  }
}
```

- 최대한 단순히 설명하기 위해, 실제 쓰레드는 사용하지 않았다.
- ThreadA가 사용자A 코드를 호출하고 ThreadB가 사용자B 코드를 호출한다 가정하자.
- StatefulService 의 price 필드는 공유되는 필드인데, 특정 클라이언트가 값을 변경한다.
- 사용자A의 주문금액은 10000원이 되어야 하는데, 20000원이라는 결과가 나왔다.
- 실무에서 이런 경우 발생 시 해결하기 어려운 큰 문제
  따라서 공유필드는 조심하고 스프링 빈은 항상 무상태(stateless)로 설계!

<br>

## 5. @Configuration과 싱글톤

```java
@Configuration
public class AppConfig {

  @Bean
  public MemberService memberService() {
    return new MemberServiceImpl(memberRepository()); // new MemoryMemberReposiotry 생성
  }

  @Bean
  public OrderService orderService() { 
    return new OrderServiceImpl(memberRepository(), discountPolicy()); // new MemoryMemberRepository 생성 -> 문제발생?
  }
  
  @Bean
  public MemberRepository memberRepository() {
    return new MemoryMemberRepository();
  }
  ...
}
```

- memberService 빈을 만드는 코드를 보면 memberRepository() 를 호출
> 이 메서드를 호출하면 new MemoryMemberRepository() 를 호출
- orderService 빈을 만드는 코드도 동일하게 memberRepository() 를 호출
> 이 메서드를 호출하면 new MemoryMemberRepository() 를 호출
- 결과적으로 각각 다른 2개의 MemoryMemberRepository 가 생성되면서 싱글톤이 깨지는 것 처럼 보인다.
- 스프링 컨테이너는 이 문제를 어떻게 해결할까?

<br>

### 검증 용도의 코드 추가

```java
public class MemberServiceImpl implements MemberService {

  private final MemberRepository memberRepository;

  //테스트 용도
  public MemberRepository getMemberRepository() { 
    return memberRepository;
  }
}

public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository;

  //테스트 용도
  public MemberRepository getMemberRepository() { 
    return memberRepository;
  }
}
```

테스트를 위해 MemberRepository를 조회할 수 있는 기능을 추가한다. 기능 검증을 위해 잠깐 사용하는 것이니 인터페이스에 조회기능까지 추가 X

<br>

#### 테스트 코드

```java
@Test
void configurationTest() { 
  ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class); 
  MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);

  OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
  MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

  //모두 같은 인스턴스를 참고하고 있다. System.out.println("memberService -> memberRepository = " +
  memberService.getMemberRepository()); 
  System.out.println("orderService -> memberRepository	= " + orderService.getMemberRepository()); System.out.println("memberRepository = " + memberRepository);

  //모두 같은 인스턴스를 참고하고 있다. assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);
  assertThat(orderService.getMemberRepository()).isSameAs(memberRepository);

}
```

- 확인해보면 memberRepository 인스턴스는 모두 같은 인스턴스가 공유되어 사용된다.
- 자바 코드를 보면 2번 new MemoryMemberRepository 호출해서 다른 인스턴스가 생성해야 함
  > 제대로 호출이 되었는지 로그를 확인.

### AppConfig에 호출 로그 남김

```java
@Bean
public MemberService memberService() {
  //1번
  System.out.println("call AppConfig.memberService"); 
  return new MemberServiceImpl(memberRepository());
}

@Bean
public OrderService orderService() {
  //1번
  System.out.println("call AppConfig.orderService"); 
  return new OrderServiceImpl(memberRepository(), discountPolicy());
}

@Bean
public MemberRepository memberRepository() {
  //2번? 3번?
  System.out.println("call AppConfig.memberRepository"); 
  return new MemoryMemberRepository();
}

@Bean
public DiscountPolicy discountPolicy() { 
  return new RateDiscountPolicy();
}
```

- 예상
  - 스프링 컨테이너가 각각 @Bean을 호출해서 스프링 빈을 생성
    1. 최초에 스프링 컨테이너가 스프링 빈에 등록하기 위해 @Bean이 붙어있는 memberRepository() 호출
    2. memberService()에서 memberRepository() 호출
    3. orderService()에서 memberRepository() 호출

- 그런데 출력 결과는 모두 1번만 호출

```bash
call AppConfig.memberService
call AppConfig.memberRepository
call AppConfig.orderService
```
<br>

## 6. @Configuration과 바이트코드 조작의 마법

스프링 컨테이너는 싱글톤 레지스트리이므로 스프링 빈이 싱클톤 보장해야 함
하지만 기존의 자바 코드 동작 형식을 바꿀순 없음.(코드로 보면 3번 호출이 맞다)
-> 그래서 스프링은 클래스의 바이트코드를 조작하는 라이브러리를 사용
-> @Configuration 을 적용한 AppConfig이 핵심

AnnotationConfigApplicationContext 에 파라미터로 넘긴 값은 스프링 빈으로 등록된다.
-> AppConfig 도 스프링 빈

<br>

### 클래스 정보를 출력 코드

```java
@Test
void configurationDeep() { 
  ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

  //AppConfig도 스프링 빈으로 등록된다.
  AppConfig bean = ac.getBean(AppConfig.class);
  System.out.println("bean = " + bean.getClass());
  //출력: bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70
}
```
<br>

### 출력문

- 실제 출력문

```bash
bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70
```

- 순수한 클래스

```bash
 class hello.core.AppConfig
```

- 클래스 명에 xxxCGLIB가 붙음
  - 내가 만든 클래스가 아니라 스프링이 CGLIB라는 바이트코드 조작 라이브러리를 사용해서 AppConfig 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한 것
- 그 임의의 다른 클래스가 바로 싱글톤이 보장되도록 해준다. 아마도 다음과 같이 바이트 코드를 조작해서 작성되어 있을 것이다.(실제로는 CGLIB의 내부 기술을 사용하는데 매우 복잡하다.)

<br>

### AppConfig@CGLIB 예상 코드
```java
@Bean
public MemberRepository memberRepository() {

  if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) { 
    return 기존의 스프링 컨테이너에서 찾아서 반환;
  } else { //스프링 컨테이너에 없으면
    기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록
    return 반환
  }
}
```
- @Bean이 붙은 메서드마다 이미 스프링 빈이 존재하면 기존에 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.
  -> 싱글톤이 보장
> 참고 AppConfig@CGLIB는 AppConfig의 자식 타입이므로, AppConfig 타입으로 조회 할 수 있다.

<br>

### @Configuration 을 적용하지 않고, @Bean 만 적용 시

- AppConfig가 CGLIB 기술 없이 순수한 AppConfig로 스프링 빈에 등록
- 출력 결과를 통해서 MemberRepository가 총 3번 호출된 것을 알 수 있다. 
- 1번은 @Bean에 의해 스프링 컨테이너에 등록하기 위해서이고, 2번은 각각 memberRepository() 를 호출하면서 발생한 코드다.

<br>

### 인스턴스가 같은지 테스트 결과

```bash
memberService -> memberRepository = hello.core.member.MemoryMemberRepository@6239aba6 
orderService -> memberRepository	= hello.core.member.MemoryMemberRepository@3e6104fc
memberRepository = hello.core.member.MemoryMemberRepository@12359a82
```

- 각각 다 다른 MemoryMemberRepository 인스턴스를 가짐

<br>

### 정리

- @Bean만 사용해도 스프링 빈으로 등록되지만, 싱글톤을 보장하지 않음
- memberRepository() 처럼 의존관계 주입이 필요해서 메서드를 직접 호출할 때 싱글톤을 보장하지 않음
- 스프링 설정 정보는 항상 @Configuration 사용 필수
