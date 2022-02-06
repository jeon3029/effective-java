# 객체 생성과 파괴2

## Item3 : private 생성자나 enum 으로 싱글턴을 보장하라

클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트가 테스트하기 어렵다
- 싱글턴 인스턴스를 가짜(mock)구현으로 대체할 수 없기 때문이다.

- 1) public static final 방식
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
}
```
- private 생성자는 초기화 할 때 딱 한번만 호출된다
- 해당 클래스가 싱글턴임을 api 에 명백히 드러남
- 예외는 리플렉션 API 인 `AccessibleObject.setAccessible` 을 이용해 private 생성자를 호출 할 수 있다.

- 2) staticfactory 방식
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }
}
```
- 리플렉션 예외는 똑같이 적용
- 장점
  - 추후 싱글턴이 아니게 변환 가능
  - 제네릭 싱글턴 팩터리로 만들 수 있다(item30)
  - 팩터리의 메서드 참조를 공급자로 사용할 수 있다
    - Elvis::getInstance 를 Supplier<Elvis> 로 사용
- 직렬화시 readResolve 메서드를 제공해야 함
  - 이렇게 하지 않으면 역직렬화시마다 새로운 인스턴스가 생김

```java
private Object readResolve(){
    return INSTANCE;//가짜 Elvis 는 gc 가 처리
}
```

- 3) 열거형으로 선언 - 바람직한 방법
```java
public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() {...}
}
```
- public field 방식과 비슷하지만 간결하고 추가 노력없지 직렬화 가능
- 대부분의 상황에서는 원소가 하나뿐인 열거타입의 싱글턴을 만드는게 좋다.
- 단 만들려는 싱글턴이 enum 이되의 클래스를 상송해야 한다면 사용할 수 없다.

## Item4 : 인스턴스화를 막으려거든 Privae 생성자를 사용하라

가끔 정정 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 떄가 있다
- (예시)
  - java.lang.Math
  - java.util.Arrays
  - java.util.Collections

- 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.
  - 하위 클래스를 만들어 인스턴스화 가능
- private 생성자를 추가하여 인스턴스화를 막는다.
  - 상속을 불가능하게 한다(명시적이든 묵시적이든 상위 클래스의 생성자를 호출하게 되는데 private 으로 되어있어 막힌다)

```java
public class UtilityClass {
    private UtilityClass() {
        throw new AssertionError();
    }
}
```

## Item5 : 자원을 직접 명시하지 말고 의존객체 주입을 사용하라


## Item6 : 불필요한 객체 생성을 피하라

## Item7 : 다 쓴 객체 참조를 해제하라

## Item8 : finalizer 와 cleaner 사용을 피하라

## Item9 : try-finally 보다는 try-with-resources 를 사용하라

- 회수가 필요한 리소스를 사용할 때 필수이다(예외는 없다)