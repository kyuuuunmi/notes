---
title: External Property Settings in Spring VS Spring Boot
description: 외부에 선언한 Property 값들을 어떻게 어플리케이션으로 불러올까?
---

Spring 과 Spring boot에서 Property value를 불러오는 부분은 초기 프로젝트 세팅 시 필수적이지만 항상 헷갈리곤 한다. 궁금했던 내용들을 정리해 본다.


## Spring 에서 외부 Property 설정하기

### 1. `PropertySourcesPlaceholderConfigurer` 사용하여 변수를 실제 설정 값으로 대체하기

수동 변환 방법으로, 프로퍼티 치환자(`placeholder`)를 사용하는 방법이다.

`PropertySourcesPlaceholderConfigurer`은 `BeanFactoryPostProcessor`의 구현체로서,
property 파일을 읽어와서 변수로 선언한 곳에 실제 값을 대체하는 것으로 동작한다.

```java
@Configuration
@PropertySource("my.properties")
public class SpringConfig {
  @Bean
  public static PropertySourcesPlaceholderConfigurer placeholderConfigurer() {
    return new PropertySourcesPlaceholderConfigurer();
  }
}
```

```java

// 필드 수동 주입
@Value(${db.url})
private String dbUrl;

...

```


### 2. `PropertiesFactoryBean` 사용하여 설정 값을 `Properties` 인스턴스로 접근하기

[`PropertiesFactoryBean`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/PropertiesFactoryBean.html) 을 선언하여, 파일 또는 로컬의 설정 값을 bean으로 등록시키는 방법이 있다.

해당 방법은 classpath에 위치한 설정 파일 내 값들을 `Properties` instance로 접근 가능하게 해준다.

```xml
<bean id="myProps" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
  <property name="location">
		<value>classpath:my.properties</value>
	</property>
</bean>
```

`myProps` bean 에서 특정 property에 접근하기 위해서 아래와 같이 접근한다.

```java

// #{...} : 능동 주입
// SpEL으로 myProps Map 객체(Properties)의 get() 메소드를 통해 db.url 필드에 접근한다.
@Value(#{myProps[db.url]})
private String dbUrl;

...

```


> `@Value` 어노테이션을 사용할 때, `${..}`와 `#{..}`의 차이는 ?  
`${..}`의 단점은 수동 주입이기 때문에, 필드에 매칭되는 키 값이 존재하지 않아도 어플리케이션 동작 시 아무런 경고도 나타나지 않는다. 그래서 의도하지 않은 오타로 키 값을 선언했을 때 해당되는 값이 존재하지 않아도 아무런 에러를 일으키지 않는다. `#{..}`을 사용한 SpEL 의 장점은 능동적인 주입으로 선언된 `Properties` 빈이 없으면 에러를 발생시킨다. 물론, 키에 해당하는 값이 없으면 null인 경우가 나타날 수 있다. <br>
[Spring Expression Language (SpEL) with @Value: dollar vs. hash ($ vs. #)](https://stackoverflow.com/questions/5322632/spring-expression-language-spel-with-value-dollar-vs-hash-vs), [Spring 3.0 (59) 프로퍼티 파일 이용하기 - placeholder vs SpEL](http://toby.epril.com/?p=968)



## Spring boot 에서 외부 Property 설정하기

Spring boot는 Type-Safe한 설정을 권하는 가이드가 있다.  
[Type-safe Configuration Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html#boot-features-external-config-typesafe-configuration-properties)

Property 값들에 맞는 구조의 POJO를 선언하여 사용해서 유효성 검사를 강제로 하게끔 하는데,

`Relaxed binding` ('context-path' 라는 key가 'contextPath' 필드로 자동으로 바인딩 됨) 기능과 `Merging Complex Types` (list/map 타입 등 복수 필드 타입 지원) 기능을 함께 제공한다.


```java
...

@ConfigurationProperties("db")
public class MyProperties {

	private String url;

	private String password;

	public void setUrl(String url) { ... }

...
}

```

사용은 객체의 필드에 직접 접근하거나 `get()` 메소드를 호출한다.  

```java
...

@Autowired
private MyProperties properties;
...

  public void test(){

    String url = properties.getUrl();

    ...
  }
...

```





## 참고
- [Class PropertiesFactoryBean](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/PropertiesFactoryBean.html)
- [propertiesfactorybean vs propertyplaceholderconfigurer spring?](https://stackoverflow.com/questions/20353999/propertiesfactorybean-vs-propertyplaceholderconfigurer-spring)
- [springframework:propertysource](https://kwonnam.pe.kr/wiki/springframework/propertysource)
- [Spring Expression Language (SpEL) with @Value: dollar vs. hash ($ vs. #)](https://stackoverflow.com/questions/5322632/spring-expression-language-spel-with-value-dollar-vs-hash-vs)
- [BeanFactoryPostProcessor VS  BeanFactoryPostProcessor](http://wonwoo.ml/index.php/post/899)
- [Spring 3.0 (59) 프로퍼티 파일 이용하기 - placeholder vs SpEL](http://toby.epril.com/?p=968)
