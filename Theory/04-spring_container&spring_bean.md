# 스프링 컨테이너와 스프링 빈

1. 스프링 컨테이너 생성
2. 컨테이너에 등록된 모든 빈 조회
3. 스프링 빈 조회 - 기본
4. 스프링 빈 조회 - 동일한 타입이 둘 이상
5. 스프링 빈 조회 - 상속 관계
6. BeanFactory와 ApplicationContext
7. 다양한 설정 형식 지원 - 자바 코드, XML
8. 스프링 빈 설정 메타 정보 - BeanDefinition

## 1. 스프링 컨테이너 생성

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```

- ApplicationContext를 스프링 컨테이너라 한다.
- ApplicationContext 는 인터페이스이다. -> 다형성 적용
- 스프링 컨테이너는 XML, 애노테이션 기반의 자바 설정 클래스 생성 가능.
  > 직전에 AppConfig 를 사용했던 방식이 애노테이션 기반의 자바 설정 클래스로 스프링 컨테이너를 만든 것이다.
- new AnnotationConfigApplicationContext(AppConfig.class);
  > 이 클래스는 ApplicationContext 인터페이스의 구현체이다.

> 원래는
> BeanFactory, ApplicationContext를 구분해서 스프링 컨테이너를 부른다.
> 하지만 일반적으로는 ApplicationContext 를 스프링 컨테이너라 한다.

### 스프링 컨테이너의 생성 과정

1. 스프링 컨테이너 생성

```java
new AnnotationConfigApplicationContext(AppConfig.class)
```

생성 시 비어있는 스프링 빈 저장소(빈 이름, 빈 객체)가 생성됨
스프링 컨테이너를 생성할 때는 구성 정보(ex. AppConfig.class)를 지정해주어야 한다.

2. 스프링 빈 등록

스프링 컨테이너는 파라미터(@Bean)로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록함

- 빈 이름은 메서드 이름을 사용한다.
- 빈 이름을 직접 부여할 수 도 있다.
  > @Bean(name="memberService2")  
  > 주의: 빈 이름은 항상 다른 이름을 부여해야 한다.(중복 불가)

3. 스프링 빈 의존관계 설정 - 준비

- 스프링 컨터이너에 빈 객체 등록 완료한 상태

4. 스프링 빈 의존관계 설정 - 완료

- 스프링 컨테이너는 설정 정보를 참고하여 의존관계를 주입(DI)한다.
- 단순히 자바 코드를 호출하는 것이 아님.(싱글톤 컨테이너 참고)
- 동적인 의존 관계가 형성됨.

## 2. 컨테이너에 등록된 모든 빈 조회

스프링 컨테이너에 실제 스프링 빈들이 잘 등록 되었는지 확인해보자.

- 내가 등록한 빈만 조회

### 예제 코드

```java
@Test
@DisplayName("애플리케이션 빈 출력하기")
void findApplicationBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for (String beanDefinitionName: beanDefinitionNames) {
        BeanDefinition beanDefinition  = ac.getBeanDefinition(beanDefinitionName);
        // Role ROLE_APPLICATION : 직접 등록한 애플리케이션 빈
        // Role ROLE_INFRASTRUCTURE : 스프링이 내부에서 사용하는 빈
        if (beanDefinition.getRole()==BeanDefinition.ROLE_APPLICATION) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("name = "+beanDefinitionName+"object = "+bean);
        }
    }
}
```

## 3. 스프링 빈 조회 - 기본

- 스프링 빈을 찾는 가장 기본적인 조회 방법

  - ac.getBean(타입)
  - ac.getBean(빈이름, 타입

- 조회 대상 스프링 빈이 없으면 예외 발생

```bash
NoSuchBeanDefinitionException: No bean named 'xxxxx' available
```

### 예제 코드

```java
class ApplicationContextBasicFindTest {
  AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
  @Test
  @DisplayName("빈 이름으로 조회")
  void findBeanByName() {
    MemberService memberService = ac.getBean("memberService", MemberService.class);
    assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
  }

  @Test
  @DisplayName("이름 없이 타입만으로 조회")
  void findBeanByType() {
    MemberService memberService = ac.getBean(MemberService.class);
    assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
  }

  // 변경 시 유연성 떨어짐
  @Test
  @DisplayName("구체 타입으로 조회")
  void findBeanByName2() {
    MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
    assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
  }

  @Test
  @DisplayName("빈 이름으로 조회X")
  void findBeanByNameX() {
    //ac.getBean("xxxxx", MemberService.class); Assertions.assertThrows(NoSuchBeanDefinitionException.class, () ->
    ac.getBean("xxxxx", MemberService.class));
  }
}
```

## 4. 스프링 빈 조회 - 동일한 타입이 둘 이상

- 타입으로 조회시 같은 타입의 스프링 빈이 둘 이상이면 오류가 발생한다.
- 이때는 빈 이름을 지정하자.

### 예제 코드

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);

  @Test
  @DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 중복 오류가 발생한다")
  void findBeanByTypeDuplicate() {
  //DiscountPolicy bean = ac.getBean(MemberRepository.class); assertThrows(NoUniqueBeanDefinitionException.class, () ->
    ac.getBean(MemberRepository.class));
  }

  @Test
  @DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 빈 이름을 지정하면 된다")
  void findBeanByName() {
    MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
    assertThat(memberRepository).isInstanceOf(MemberRepository.class);
  }

  // ac.getBeansOfType() : 해당 타입의 모든 빈을 조회할 수 있다.
  @Test
  @DisplayName("특정 타입을 모두 조회하기")
  void findAllBeanByType() {
    Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
    for (String key : beansOfType.keySet()) {
      System.out.println("key = " + key + " value = " + beansOfType.get(key));
  }

  System.out.println("beansOfType = " + beansOfType); assertThat(beansOfType.size()).isEqualTo(2);
}

  @Configuration
  static class SameBeanConfig {

  @Bean
  public MemberRepository memberRepository1() {
    return new MemoryMemberRepository();
  }

  @Bean
  public MemberRepository memberRepository2() {
    return new MemoryMemberRepository();
  }
}
```

## 5. 스프링 빈 조회 - 상속 관계

- 부모 타입으로 조회하면 무조건 자식들도 조회된다.
- 그래서 모든 자바 객체의 최고 부모인 Object 타입으로 조회하면, 모든 스프링 빈을 조회한다.

### 예제 코드

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

  @Test
  @DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면, 중복 오류가 발생한다")
  void findBeanByParentTypeDuplicate() {
  //DiscountPolicy bean = ac.getBean(DiscountPolicy.class); assertThrows(NoUniqueBeanDefinitionException.class, () ->
    ac.getBean(DiscountPolicy.class));
  }

  @Test
  @DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면, 빈 이름을 지정하면 된다")
  void findBeanByParentTypeBeanName() {
    DiscountPolicy rateDiscountPolicy = ac.getBean("rateDiscountPolicy", DiscountPolicy.class);
    assertThat(rateDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
  }

  @Test
  @DisplayName("특정 하위 타입으로 조회")
  void findBeanBySubType() {
    RateDiscountPolicy bean = ac.getBean(RateDiscountPolicy.class);
    assertThat(bean).isInstanceOf(RateDiscountPolicy.class);
  }

  @Test
  @DisplayName("부모 타입으로 모두 조회하기")
  void findAllBeanByParentType() {
    Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);
    assertThat(beansOfType.size()).isEqualTo(2);
    for (String key : beansOfType.keySet()) {
      System.out.println("key = " + key + " value=" + beansOfType.get(key));
    }
  }

  // 스프링 내 모든 객체가 출력된다
  @Test
  @DisplayName("부모 타입으로 모두 조회하기 - Object")
  void findAllBeanByObjectType() {
    Map<String, Object> beansOfType = ac.getBeansOfType(Object.class);
    for (String key : beansOfType.keySet()) {
      System.out.println("key = " + key + " value=" + beansOfType.get(key));
    }
  }

  @Configuration
  static class TestConfig {

    @Bean
    public DiscountPolicy rateDiscountPolicy() {
      return new RateDiscountPolicy();
    }

    @Bean
    public DiscountPolicy fixDiscountPolicy() {
      return new FixDiscountPolicy();
    }
  }
```

## 6. BeanFactory와 ApplicationContext

### BeanFactory

- 스프링 컨테이너의 최상위 인터페이스
- 스프링 빈을 관리, 조회하는 역할을 담당
- getBean()을 제공
- 지금까지 우리가 사용했던 대부분의 기능은 BeanFactory가 제공하는 기능

### ApplicationContext

- BeanFactory 기능을 모두 상속받아서 제공
- 빈을 관리하고 검색하는 기능을 BeanFactory가 제공해주는데, 그러면 둘의 차이가 뭘까?
  > 빈을 관리하고 조회하는 기능은 물론이고, 수많은 부가기능을 제공

#### ApplicatonContext가 제공하는 부가기능

- 메시지소스를 활용한 국제화 기능
  > 예를 들어서 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력
- 환경변수
  > 로컬, 개발, 운영등을 구분해서 처리
- 애플리케이션 이벤트
  > 이벤트를 발행하고 구독하는 모델을 편리하게 지원
- 편리한 리소스 조회
  > 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회

#### 정리

- ApplicationContext는 BeanFactory의 기능을 상속, 빈 관리기능 + 편리한 부가 기능을 제공
- BeanFactory를 직접 사용할 일은 거의 없다.
  -> 부가기능이 포함된 ApplicationContext를 사용한다.
- BeanFactory나 ApplicationContext를 스프링 컨테이너라 한다.

## 7. 다양한 설정 형식 지원 - 자바 코드, XML

- 스프링 컨테이너는 다양한 형식의 설정 정보를 받아드릴 수 있도록 유연하게 설계됨.
  > 자바 코드, XML, Groovy 등등

### 애노테이션 기반 자바 코드 설정 사용

> 지금까지 했던 것이다.

```java
new AnnotationConfigApplicationContext(AppConfig.class)
```

AnnotationConfigApplicationContext 클래스를 사용하면서 자바 코드로된 설정 정보를 넘기면 된다.

### XML 설정 사용

최근 스프링 부트를 많이 사용하면서 XML기반의 설정은 잘 사용하지 않지만 아직 많은 레거시 프로젝트들이 XML로 되어 있다.  
또 XML을 사용하면 컴파일 없이 빈 설정 정보를 변경할 수 있는 장점도 있으므로 배우고 넘어가자.

> GenericXmlApplicationContext 를 사용하면서 xml 설정 파일을 넘기면 된다.

```java
ApplicationContext ac = new GenericXmlApplicationContext("appConfig.xml");
MemberService memberService = ac.getBean("memberService", MemberService.class);
```

### xml 기반의 스프링 빈 설정 정보

src/main/resources/appConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.springframework.org/schema/beans http://
www.springframework.org/schema/beans/spring-beans.xsd">

<bean id="memberService" class="hello.core.member.MemberServiceImpl">
  <constructor-arg name="memberRepository" ref="memberRepository" />
</bean>

<bean id="memberRepository" class="hello.core.member.MemoryMemberRepository" />

<bean id="orderService" class="hello.core.order.OrderServiceImpl">
  <constructor-arg name="memberRepository" ref="memberRepository" />
  <constructor-arg name="discountPolicy" ref="discountPolicy" />
</bean>

<bean id="discountPolicy" class="hello.core.discount.RateDiscountPolicy" />
```

- xml 기반의 appConfig.xml 스프링 설정 정보와 자바 코드로 된 AppConfig.java 설정 정보를 비교해보면 거의 비슷하다.  
  필요하면 스프링 [공식 레퍼런스 문서](https://spring.io/projects/spring-framework)를 확인하자.

## 8. 스프링 빈 설정 메타 정보 - BeanDefinition

- 스프링이 다양한 설정 형식을 지원할 수 있는 이유?
  -> 그 중심에는 BeanDefinition 이라는 추상화가 있다.
- 즉, 역할과 구현을 개념적으로 나눈 것이다!
  > 스프링 컨테이너는 자바 코드인지, XML인지 몰라도 된다. 오직 BeanDefinition만 알면 된다.  
  > XML을 읽어서 BeanDefinition을 만들면 된다.
  > 자바 코드를 읽어서 BeanDefinition을 만들면 된다.
- BeanDefinition 을 빈 설정 메타정보라 한다.
  - @Bean, <bean> 당 각각 하나씩 메타 정보가 생성된다.
  - 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성한다.

### 더 깊이 설명

- AnnotationConfigApplicationContext는 AnnotatedBeanDefinitionReader를 사용해서
  AppConfig.class 를 읽고 BeanDefinition 을 생성한다.
- GenericXmlApplicationContext 는 XmlBeanDefinitionReader 를 사용해서 appConfig.xml정보를 읽고 BeanDefinition 을 생성한다.
  -> 따라서, 새로운 형식의 설정 정보가 추가되면, XxxBeanDefinitionReader를 만들어서 BeanDefinition 을 설정하고 생성하면 된다.

### BeanDefinition 살펴보기

#### BeanDefinition 정보

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
//	GenericXmlApplicationContext ac = new GenericXmlApplicationContext("appConfig.xml");

  @Test
  @DisplayName("빈 설정 메타정보 확인")
  void findApplicationBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for (String beanDefinitionName : beanDefinitionNames) {
      BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);
      if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
        System.out.println("beanDefinitionName" + beanDefinitionName + " beanDefinition = " + beanDefinition);
      }
    }
  }
```

- BeanClassName: 생성할 빈의 클래스 명(자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음)
- factoryBeanName: 팩토리 역할의 빈을 사용할 경우 이름, 예) appConfig
- factoryMethodName: 빈을 생성할 팩토리 메서드 지정, 예) memberService
- Scope: 싱글톤(기본값)
- lazyInit: 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때 까지 최대한 생성을 지연처리 하는지 여부
- InitMethodName: 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 명
- DestroyMethodName: 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
- Constructor arguments, Properties: 의존관계 주입에서 사용한다. (자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음)

#### 정리

- BeanDefinition을 직접 생성해서 스프링 컨테이너에 등록할 수도 있다.
- 스프링이 다양한 형태의 설정 정보를 BeanDefinition으로 추상화해서 사용하는 것 정도만 이해
- 가끔 스프링 코드나 스프링 관련 오픈 소스의 코드를 볼 때, BeanDefinition이 위의 개념임을 인지하자
