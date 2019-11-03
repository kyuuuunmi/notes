---
title: Supper Token Type & TypeReference
description: Generic을 더 유연하게, Type-Safe하게 쓰는 방법
---

Super Token Type, Generic을 더 유연하게, Type-Safe 하게 쓰는 방법을 알아보자.


## 문제

다른 서비스와 통신하기 위해 아래와 같은 공통의 메세지 타입 `Message`를 정의했다.
`integer` 타입의 number을 명시적으로 선언했고, 사용처에 따라 동적으로 타입이 정의될 수 있도록 제네릭 타입(`T`)의 data를 추가로 정의하였다.

```java
public class Message<T> {
        int number;
        T data;
}
```

json 으로 받아온 `String` 타입의 value를 `Message`로 deserialize 하기 위해 `objectMapper`를 사용하여 객체로 매핑을 시도했다. 그런데 아래와 같이 선언하는 것은 불가능했다.

```java

// Message<Mail>.class 와 같은 선언은 불가능하다.
Message<Mail> mailMessage = objectMapper.readValue(jsonValue, Message<Mail>.class);

```

`Message` 객체의 `data` 타입을 동적으로 형변환하려던 것이었는데 실패했다. 어떻게 하면 제네릭 타입인 `Meesage<Mail>`을 선언할 수 있을까? 그리고 제네릭 타입이 포함된 DTO를 어떤 방법으로 최소한의 코드로 한 번에 매핑할 수 있을까?


## 해결법

해결 방법부터 먼저 제시하자면, `SuperTypeToken` 패턴을 사용한다.  

**제네릭 정보가 컴파일 할 때 모두 지워지지만, 제네릭 클래스를 정의한 후에 그 제네릭 클래스를 상속받으면 런타임시에는 제네릭 정보를 가져올 수 있다.**

자세한 설명은 아래에서 계속하고, 해결안을 먼저 보자.

```java

Message<Mail> mailMessage = objectMapper.readValue(jsonValue, TypeReference<Message<Mail>>(){});

```

Jackson에서는 `TypeReference` 을 제공해주는데, 해당 클래스는 제네릭 타입으로 선언된 **실제 타입 정보**(`Message<Mail>`)를 조회할 수 있다. 즉, `jsonValue`를 `Message<Mail>`타입으로 정확히 형변환하여 매핑이 가능(`물론 string으로 제시된 value가 깨지지 않는 조건이 필수로 선행된다.`)하다 !



## TypeToken 이란 ? 그리고 한계


SuperTypeToken은 둘째치고, TypeToken은 무엇일까?

먼저 제네릭에 대해 되짚어보자.

제네릭은 JDK5부터 추가된 문법으로 타입 정보를 인스턴스가 생성되는 시점, 즉 동적으로 정의할 수 있다.  
컴파일 시에 컴파일러가 Type Casting을 해주고, 명시적으로 타입을 지정한다는 점에서 Type-Safe한 방식이다.  

예)

```java
// 제네릭은 이와 같은 실수를 막는다.
Object o = "String";
Integer i = (Integer)o;  // String 타입인데, Integer 로 형변환을 시도함 -> Runtime 오류
System.out.println(i);

```

여기서 이야기 할 수 있는 것이 **TypeToken** 이다.

특정 타입의 클래스 정보를 넘겨서 타입 안정성을 꽤하는 방법을 TypeToken이라고 하며, Class Literal(`String.class`, `Integer.class` ...)이 Type Token으로서 사용된다.


```java
// Apple.class 클래스 리터럴이 타입 토큰 파라미터로 readValue() 메소드에 전달되었다.
Apple a = objectMapper.readValue(json, Apple.class);
```


하지만, 맨 처음 문제점으로 제시한 것 처럼 제네릭 타입을 포함한 타입 정보는 넘길 수 없다.

```java

List<Apple>.class 과 같은 선언은 불가

```


## 대안, 그리고 SuperTypeToken



SuperTypeToken은 리플렉션 기능과 `Class`에서 제공하는 `getGenericSuperclass()` 메소드를 사용한 방법이다.  Neal Gafter가 최초로 이 기법을 제시했다고 한다.


헤딩 메소드는 부모 클래스의 타입을 가져오는 메소드로, 정적으로 정의된 제네릭 타입(`Class<List<Apple>>`)에 접근할 수 있는 실마리를 제공한다. `getGenericSuperclass()`로 얻은 부모의 타입을 `ParameterizedType`으로 형변환 시키면 실제 정의된 타입을 알 수 있게 된다.


> List, Map등의 변수에 실제 할당된 Generic Type 을 가져오기 위해 ParameterizedType를 사용한다.  
> 참고 ) [Interface ParameterizedType](https://docs.oracle.com/javase/7/docs/api/java/lang/reflect/ParameterizedType.html)



즉, 어떤 객체 `a`의 바로 위의 수퍼 클래스가 `List<Apple>` 이라는 파라미터를 사용하고 있는 ParameterizedType이면, `a.getClass().getGenericSuperclass()`는 `List<Apple>` 정보가 포함되어 있는 타입을 반환한다.


아래와 같은 클래스를 선언해보자.

```java

abstract class TypeReference<T> {
    Type type; // Generic 형태를 포함한 정보를 저장

    public TypeReference() {
      // 부모 타입에 접근하여, 실제 타입을 조회한다.
        Type sType = getClass().getGenericSuperclass();
        if (sType instanceof ParameterizedType) {
            this.type = ((ParameterizedTypesType).getActualTypeArguments()[0];
        } else throw new RuntimeException();
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass().getSuperclass() !o.getClass().getSuperclass()) return false;

        TypeReference<?> that = (TypeReference<?>) o;

        return type.equals(that.type);
    }

    @Override
    public int hashCode() {
        return type.hashCode();
    }
}


```

부모 타입에 접근한 뒤 실제 타입을 조회하여 Generic 형태를 포함한 정보를 저장한 클래스이다.
부모 타입을 반드시 필요로 하기 때문에, Anonymous Class 로 선언하여 다음과 같이 사용한다.


```java
class TypesafeMap {
    Map<TypeReference<?>, Object> map = new HashMap<>();

    <T> void put(TypeReference<T> tr, T value) {
        map.put(tr, value);
    }

    <T> T get(TypeReference<T> tr) {
        if (tr.type instanceof Class<?>)
            return ((Class<T>) tr.type).cast(map.get(tr));
        else
            return ((Class<T>) ((ParameterizedType) tr.type).getRawType()).cast(map.get(tr));
    }
}

public static void main(String args[]){

  TypesafeMap m = new TypesafeMap();
  // Anonymous Class 선언으로 부모 타입에 접근 가능하도록 한다.
  m.put(new TypeReference<Integer>() { }, 1);
  m.put(new TypeReference<String>() { }, "String");
  m.put(new TypeReference<List<Integer>>() { }, Arrays.asList(1, 2, 3));

}
```


## 얻은 것

예시에서는 `TypeReference`를 직접 구현하였지만, 스프링을 포함한 여러 라이브러리에서 해당 유틸을 제공하고 있다. ([Spring - `ParameterizedType`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/core/ParameterizedTypeReference.html)).


SuperTypeToken 방식 적용으로 제네릭을 사용함으로써 얻는 Type-Safe 한 형변환의 장점을 잃지 않게 되었고, 제네릭이 포함된 DTO 변환시에도 간결한 코드를 유지할 수 있게 되었다!



## 참고
- [Super Type Tokens - Neal Gafter's blog](http://gafter.blogspot.com/2006/12/super-type-tokens.html)
- [클래스 리터럴, 타입 토큰, 수퍼 타입 토큰](https://homoefficio.github.io/2016/11/30/%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%A6%AC%ED%84%B0%EB%9F%B4-%ED%83%80%EC%9E%85-%ED%86%A0%ED%81%B0-%EC%88%98%ED%8D%BC-%ED%83%80%EC%9E%85-%ED%86%A0%ED%81%B0/)
- [토비의 봄 TV 2회 - 수퍼 타입 토큰](https://youtu.be/01sdXvZSjcI)
