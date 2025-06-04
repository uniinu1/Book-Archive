# 아이템 3: private 생성자 또는 enum 타입을 사용해서 싱글톤으로 만들 것
오직 한 인스턴스만 만드는 클래스를 *싱글톤*이라 부른다. 보통 함수 같은 Stateless 객체([아이템 24](item24.md)) 또는 본질적으로 유일한 시스템 컴포넌트를 그렇게 만든다.

**싱글톤을 사용하는 클라이언트 코드를 테스트 하는게 어렵다.** 싱글톤이 인터페이스를 구현한게 아니라면 mock으로 교체하는게 어렵기 때문이다.

- 싱글톤이란?
    - 애플리케이션 전체에서 유일한 인스턴스를 갖고, SomeSingleton.getInstance() 같은 정적 메서드로 꺼내 쓰는 패턴입니다.

- 테스트하기 어려운 이유
    - 테스트할 때는 “실제 로직” 대신 “가짜(mock) 구현”을 넣어서 경로 분기를 제어하거나 외부 의존을 격리하고 싶습니다.
    - 예를 들어 DB 호출, 네트워크 I/O, 파일 시스템 접근 같은 부작용(side effect)을 제거하고 순수 로직만 단위 테스트하고 싶을 때 유용하죠.

- 인터페이스를 구현하지 않으면
    - 만약 싱글톤 클래스가 어떤 인터페이스를 구현하고 있다면, 테스트 환경에서 그 인터페이스를 구현한 가짜(mock) 객체를 만들어 SomeSingleton.setInstance(mock) 같은 방식으로 교체할 수 있습니다.
    - 그러나 인터페이스로 추상화되어 있지 않고, SomeSingleton.getInstance()가 반환하는 구체 클래스 타입만 제공한다면, 테스트 시점에 가짜 구현으로 바꿀 방법(포인트컷 설정, DI 주입 등)이 없습니다.

- 결과적으로
    - 클라이언트 코드는 항상 SomeSingleton.getInstance()로 고정된 구현체를 호출하기 때문에,
    - 테스트 환경에서 “다른 동작을 하는 싱글톤”으로 교체할 수 없어서 mocking이 불가능해집니다.


싱글톤으로 만드는 두가지 방법이 있는데, 두 방법 모두 생성자를 private 으로 만들고 publis static 멤버를 사용해서 유일한 인스턴스를 제공한다.

## final 필드

첫 번째 방법은 final 필드로 제공한다.

```java
public class Elvis {

    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

}
```

리플렉션을 사용해서 private 생성자를 호출하는 방법을 제외하면 (그 방법을 막고자 생성자 안에서 카운팅하거나 flag를 이용해서 예외를 던지게 할 수도 있지만) 생성자는 오직 최초 한번만 호출되고 Elvis는 싱글톤이 된다.

### 장점

이런 API 사용이 static 팩토리 메소드를 사용하는 방법에 비해 더 명확하고 더 간단하다. 

## static 팩토리 메소드

```java
public class Elvis {

    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

    public static Elvis getInstance() {
        return INSTANCE;
    }

}
```

### 장점

API를 변경하지 않고로 싱글톤으로 쓸지 안쓸지 변경할 수 있다. 처음엔 싱글톤으로 쓰다가 나중엔 쓰레드당 새 인스턴스를 만든다는 등 클라이언트 코드를 고치지 않고도 변경할 수 있다.

필요하다면 `Generic 싱글톤 팩토리`([아이템 30](item30.md))를 만들 수도 있다.

static 팩토리 메소드를 `Supplier<Elvis>`에 대한 `메소드 레퍼런스`로 사용할 수도 있다.

## 직렬화 (Serialization)

위에서 살펴본 두 방법 모두, 직렬화에 사용한다면 역직렬화 할 때 같은 타입의 인스턴스가 여러개 생길 수 있다. 그 문제를 해결하려면 모든 인스턴스 필드에 `transient`를 추가 (직렬화 하지 않겠다는 뜻) 하고 `readResolve` 메소드를 다음과 같이 구현하면 된다. (객체 직렬화 API의 비밀 참고)

```java
    private Object readResolve() {
        return INSTANCE;
    }

```

## Enum

직렬화/역직렬화 할 때 코딩으로 문제를 해결할 필요도 없고, 리플렉션으로 호출되는 문제도 고민할 필요없는 방법이 있다.

```java
public enum Elvis {
    INSTANCE;
}
```

코드는 좀 불편하게 느껴지지만 싱글톤을 구현하는 최선의 방법이다. 하지만 이 방법은 Enum 말고 다른 상위 클래스를 상속해야 한다면 사용할 수 없다. (하지만 인터페스는 구현할 수 있다.)

## 참고

* [객체 직렬화 API의 비밀](http://www.oracle.com/technetwork/articles/java/javaserial-1536170.html)
