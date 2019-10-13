---
title: Spring Security
description: 도대체 Spring Security 프레임워크 무엇이고, 사용했을 때 어떠한 장점이 있는가?
---


웹 서비스에 보안을 적용하는데 있어서 중요한 2가지가 있다.
`Authentication` 인증, 그리고 `Authorization` 인가(권한 부여)이다.

**Authentication** 인증
   - 너는 누구냐 ! 나는 OOO 입니다! 이것을 보세요! 이것들은 나를 증명해줍니다!
   - 어플리케이션의 사용자가 해당 사용자가 주장하는 본인이 맞는지를 확인하는 절차

**Authorization** 인가
   - 너는 무엇에 허락을 받았느냐 !
   - 인증된 주체를 권한에 매핑하고, 보호된 리소스에 대한 권한을 체크하는 것


적절한 권한을 가진 자만 해당 자원에 접근할 수 있도록 자원의 외부요청을 원천적으로 가로채는 것이 웹 보안, 그 중 인가(`Authorization`, 권한 부여)의 핵심 원칙이다.


>웹 서비스 개발 시 고려해야 할 보안 사항  
>   : 어떤 방식으로 권한을 부여할까?  
>   : 리소스에 어떻게 권한 수준을 부여하지?  
>   : 인증받은 사람은 어떻게 인증받았다는 정보를 지속적으로 유지할 수 있을까?


Spring Security는 이러한 어플리케이션의 보안의 핵심인 **인증** 과 **인가** 를 제공하는 데에 초점이 맞춰져 있고, 다른 스프링 프레임워크와 마찬가지로 요구사항에 맞춰서 쉽게 커스텀하여 확장 가능하도록 되어있다.

- 포괄적이고 확장 가능한 인증 및 인가 기능 지원
- Session Fixation, ClickJacking, Cross Site Request Forgery 등의 공격 방어

Spring Security 공식 페이지 : [https://spring.io/projects/spring-security][스프링시큐리티공식페이지]
{:.faded}


> **Session Fixation** : 유효한 유저 세션을 탈취하여 인증을 위회하는 수법  
> **ClickJacking** : 웹 사용자가 자신이 클릭하고 있다고 인지하는 것과 다른 어떤 것을 클릭하게 속이는 기법. (frame, iframe을 이용해서 사용자가 인지하지 못하게 하는 투명한 레이어를 추가하여 악성 링크로 요청을 보내도록 하는 수법 )
> **Cross Site Request Forgery** : 사용자가 현재 로그인해 있는 취약한 사이트로 악의적인 사이트에서 요청을 전송하는 공격
>
> 출처 ) [Session Fixation 공격이란?][Session Fixation 공격이란?], [X-Frame-Options이란?][X-Frame-Options이란?]


스프링 시큐리티는 서블릿 필터 레벨에서 작동한다. 리소스에 접근권한을 설정하는 방식을 intercept 방식으로 작동하는데, spring MVC가 아니더라도 방어가 가능하도록 하는데에 목적이 있어서 _내부 비즈니스 로직을 처리하는 부분과는 독립적으로 운영_ 될 수 있다.
> mvc가 아닌, DelagatingFilterProxy 을 사용


# AuthenticationManager vs ProviderManager
ProviderManager은 AuthenticationManager의 구현체로, **AuthenticationProvider** 의 인스턴스들의 체인 작업을 위임한다. AuthenticationManager와 달리 **AuthenticationProvider** 은 주어진 Authentication 객체를 지원하는지에 대한 여부를 알 수 있는 메소드를 제공한다. ProviderManager은 AuthenticationProvider의 체인에 작업을 위임시키며 여러개의 다른 인증 메카니즘을 지원한다. 만약 ProviderManager가 특정 인증 객체를 지원하지 않는다면, 해당 provider는 스킵된다.

가끔 특정 리소스 그룹 단위로 (`/api/**`) AuthenticationManager 를 가질 수 있다. 각각의 매니저들은 ProviderManager로 구현될 것이고, 공통의 부모 ProviderManager를 공유할 것이다.


![An AuthenticationManager hierarchy using ProviderManager](https://kyuuuunmi.github.io/notes/assets/img/posts/2019-03-03-Spring-Security-01.png){:.lead data-width="800" data-height="100"}
출처 : [스프링 시큐리티 공식 구조 가이드 문서][스프링 시큐리티 구조]
{:.figure}


## 결론  
- Spring Security는 보안을 적용하기 위한 인증과 권한 부여에 대한 절차를 비즈니스 로직과는 독립적으로 운영할 수 있는 기반을 마련해 준다.
- 보안에 대한 개념이 부족한 상태에서 이를 적용한다고 했을 때, 최선의 방어책이 되어 줄 수 있다.



## 참고
- [스프링 시큐리티란?][http://springmvc.egloos.com/504862]
- [Spring Security와 보안, 첫번째 이야기][http://www.nextree.co.kr/p1886/]
- [Spring handler Interceptor vs Servlet Filter][https://medium.com/@ohjongsung/spring-handlerinterceptor-vs-servlet-filter-a78a85ebc512]

[스프링시큐리티공식페이지]: https://spring.io/projects/spring-security
[Session Fixation 공격이란?]: https://reiphiel.tistory.com/entry/session-fixation-vulnerability  
[X-Frame-Options이란?]: (https://blog.one0.me/etc/X-Frame-Options-%EC%9D%B4%EB%9E%80/)
[스프링 시큐리티 구조]: https://spring.io/guides/topicals/spring-security-architecture/
