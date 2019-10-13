---
title: External Property Settings in Spring VS Spring Boot
description: 외부에 선언한 Property 값들을 어떻게 어플리케이션으로 불러올까?
---

Spring 과 Spring boot에서 Property value를 불러오는 부분은 항상 헷갈리고, 할 때마다 구글을 찾곤 했다. 정리를 해 놓으면 적어도 한 번은 구글링을 안하고도 설정 가능 하겠지 라고 바라며 궁금했던 내용들을 정리한다.


## Spring 에서 외부 Property 설정하기

[`PropertiesFactoryBean`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/PropertiesFactoryBean.html) 을 선언하여, 파일 또는 로컬의 설정 값을 bean으로 등록시키는 방법이 있다.

 즉, classpath에 위치한 설정 파일 내(`ex) **.properties`) 값들을 `Properties` instance로 접근 가능하게 해준다.

```xml
<bean id="myProps" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
		<property name="location">
			<value>classpath:my.properties</value>
		</property>
</bean>
```

```java
...

@Value(#{myProps[db.url]})
private String dbUrl;

...

```


`PropertySourcesPlaceholderConfigurer`은 `BeanFactoryPostProcessor`의 구현체로서, property 파일을 읽어와서 변수로 선언한 곳(`${somevar}`)에 실제 값을 대체하는 것으로 동작한다. `Properties` 객체로 사용하는 것은 아니다.

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
...

@Value(${db.url})
private String dbUrl;

...

```


> `@Value` 어노테이션을 사용할 때, `${..}`와 `#{..}`의 차이는 ? ([Spring Expression Language (SpEL) with @Value: dollar vs. hash ($ vs. #)](https://stackoverflow.com/questions/5322632/spring-expression-language-spel-with-value-dollar-vs-hash-vs))

## Spring boot 에서 외부 Property 설정하기

Spring boot는 Type-Safe한 설정을 권하는 가이드가 있다.

[Type-safe Configuration Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html#boot-features-external-config-typesafe-configuration-properties)

Property 값들에 맞는 구조의 POJO를 선언하여 사용해서 유효성 검사를 강제로 하게끔 하는데,

`Relaxed binding` (`context-path` 라는 속성은 `contextPath` 필드로 자동으로 바인딩 됨) 기능과 `Merging Complex Types` (list/map 타입 등 복수 필드 타입 지원 ) 기능을 함께 제공한다.


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


## 참고
- [Class PropertiesFactoryBean](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/PropertiesFactoryBean.html)
- [propertiesfactorybean vs propertyplaceholderconfigurer spring?](https://stackoverflow.com/questions/20353999/propertiesfactorybean-vs-propertyplaceholderconfigurer-spring)
- [springframework:propertysource](https://kwonnam.pe.kr/wiki/springframework/propertysource)
- [Spring Expression Language (SpEL) with @Value: dollar vs. hash ($ vs. #)](https://stackoverflow.com/questions/5322632/spring-expression-language-spel-with-value-dollar-vs-hash-vs)
- [BeanFactoryPostProcessor VS  BeanFactoryPostProcessor](http://wonwoo.ml/index.php/post/899)
