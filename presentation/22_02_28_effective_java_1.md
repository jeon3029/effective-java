# 이펙티브 자바 공부 내용 정리 자료

- `Part 1 객체 생성과 파괴`
- 첫번째 발표는 객체의 생성과 파괴(이펙티브 자바 첫번째 파트) 관련하여 진행..
- 추후에는 생각해볼만한 아이템 추려서 진행(모든 아이템 다루기에는 양이 많음)
- 한 가지 느낀것은 책을 쭉 읽는것은 그다지 도움이 되진 않았다.
  - 실제 사례에서 적용하질 못했음

## Item1 : 생성자 대신 정적 팩터리 메서드를 고려하자

### 팩터리 메서드란?

- 객체 생성역할을 하는 클래스의 메서드(생성자 X)
  
```java
public class Product{
    private String name;
    public Product(String name){
        this.name = name;
    }
    public static Product nameOf(String name){
        return new Product(name);
    }
}
```

- 장점
1. 이름을 가질 수 있음
```java
Product p1 = new Product("book");
Product p2 = Product.nameOf("pencil");
```
- 좀 더 명확함

2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 됨
- 인스턴스를 만들어두거나 생성한 인스턴스를 캐싱하여 재활용

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
//객체를 생성하는 것이 아니라 상수를 반환함
//성능상의 이점
```

3. 반환 타입의 하위타입 객체를 반환할 수 있는 능력이 있다
4. 입력 매개변수에 따라 다른 클래스의 객체를 반환할 수 있음

```java
public static Discount createDisCountProduct(String code){
    if(isCoupon(code)){
        return new Coupon(1000);
    }
    else if(isPoint(code)){
        return new Point(2000);
    }
}
class Coupon extends Discount{};
class Point extends Discount{};
//하위 클래스를 노출하지 않고 반환할 수 있다.
```
- Collections 가 45개의 구현체를 제공한다.
  - Collections 클래스를 만들지 않고 정적 팩토리 메서드를 통해 얻도록

5. 정적 팩터리 메서드를 작성하는 시점에 반환할 객체의 클래스가 없어도 됨
??
6. 객체 생성의 캡슐화

```java
public class ProductDto{
    private String name;
    private String date;
    public public static ProductDto from(Product product) {
        return new ProductDto(product.getName(),product.getDate());
    }
}
//실 사용
Product product = repositoru.getById(id);
ProductDto p1 = new ProductDto(product.getName(),product.getDate());
ProductDto p1 = ProductDto.from(product);//생성자 대체 뿐만 아니라 가독성 좋고 객체지향적으로 작성 가능
```


- 단점
1. 상속을 하려면 `public` `protected` 생성자 가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.
- Collections 의 생성자의 접근제어자가 private 임. 
2. 프로그래머가 찾기 어렵다
- 생성자 방식만큼 명확하지 않아 문서화 및 명명규칙을 따르는 방식이 필요함.

- 예시
```java
from : 매개변수 하나 받아서 객체 생성
Date d = Date.from(instant);

of : 매개변수 여러개 받아서 객체 생성
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);

valueOf : from, of 의 더 자세한 버전
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);

instance | getInstance : 인스턴스를 생성(이전에 반환한 것과 동일)
StackWalker luke = StackWalker.getInstance(options);

create | newInstance : 매번 새로운 인스턴스 임을 보장
Object newArray = Array.newInstance(classObject, arrayLen);

getType : 다른 클래스를 생성할 때
FileStore fs = Files.getFileStore(path);

newType : 다른 클래스 생성, 새로운 인스턴스
BufferedReader br = Files.newBufferedReader(path);

type : getType, newType 의 간결한 버전
List<Complaint> litany = Collections.list(legacyLitany);
```

상대적인 장단점을 이해하고 사용하자

## Item2 : 생성자에 매개 변수가 많다면 빌더를 고려하자

자신만의 기준이 중요한 것 같다. 예를들면 변소 몇개이상이면 빌더를 만든다(또는 정적 팩토리)
- 매개변수가 많으면 코드의 가독성을 현저히 떨어뜨린다.


1. telescoping constructor
```java
public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```
- 딱 봐도 관리하기 어렵게 느껴진다.(순서 바뀌거나 갯수 잘못 넣거나 등등..)

2. java beans
```
public NutritionFacts() { }
    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)     { servings = val; }
    public void setCalories(int val)     { calories = val; }
    public void setFat(int val)          { fat = val; }
    public void setSodium(int val)       { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
```
- 객체가 완성되기 전까지 일관성(consistency) 이 무너진 상태

1. 빌더패턴
```java
// 구현부
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
        { calories = val;      return this; }
        public Builder fat(int val)
        { fat = val;           return this; }
        public Builder sodium(int val)
        { sodium = val;        return this; }
        public Builder carbohydrate(int val)
        { carbohydrate = val;  return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
// 호출부
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();
```
- 쓰기 쉽고 읽기 쉬워서 장점이 많다.
- 아래의 계층적으로 설계된 구조에서 사용하기 좋다.
- 음 한가지 개인적인 궁금증이 있는데
  - 리스트 형태를 한번에 넣으려면 어떻게 하는것이 좋을까?(아이템 갯수가 몇개인지 모르는 상태)
  - Cos SendMail 관련 코드 보면서 설명

4. 계층적 빌더 패턴

- Pizza
```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }
    
    Pizza(Builder<?> buislder) {
        toppings = builder.toppings.clone(); // 아이템 50 참조
    }
}
```

- NyPizza
```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    @Override public String toString() {
        return toppings + "로 토핑한 뉴욕 피자";
    }
}
```

- Calzone
```java
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // 기본값

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }

        @Override protected Builder self() { return this; }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }

    @Override public String toString() {
        return String.format("%s로 토핑한 칼초네 피자 (소스는 %s에)",
                toppings, sauceInside ? "안" : "바깥");
    }
}
```

사용 방법
```java
NyPizza pizza = new NyPizza.Builder(SMALL)
        .addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder()
        .addTopping(HAM).sauceInside().build();
```

> 일반적으로 매개변수가 4개는 되어야 빌더 패턴 고려를 해보자

# 객체 생성과 파괴2

## Item3 : private 생성자나 enum 으로 싱글턴을 보장하라

클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트가 테스트하기 어렵다.
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
- 단 만들려는 싱글턴이 enum 이외의 클래스를 상속해야 한다면 사용할 수 없다.

## Item4 : 인스턴스화를 막으려거든 Private 생성자를 사용하라

가끔 정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 떄가 있다
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

클래스 내에서 다른 클래스를 사용할 떄 정적 변수로 사용하게 되면 유연하지 않다.
```java
public class SpellChecker{
    private static final Lexion dictionary = ...;
    private SpellChecker(){}
    public boolean isValid(String word){...}
    public List<String> suggestions(String typo){...}
}
```

```java
public class SpellChecker{
    private final Lexion dictionary = ...;
    private SpellChecker(...){}
    public static SpellChecker INSTANCE = new SpellChecker(..);
    public boolean isValid(String word){...}
    public List<String> suggestions(String typo){...}
}
```
- 유연하지 않고 테스트 하기 어렵다
- 사용하는 자원에 따라 동작이 달리지는 클래스는 정적 방식이나 싱글턴 방식이 적합하지 않다.

- 추천하는 방식 : 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식

```java
public class SpellChecker{
    private final Lexion dictionary;
    public SpellChecker(Lexion dictionary){
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    public boolean isValid(String word){...}
    public List<String> suggestions(String typo){...}
}
```

- 의존객체주입은 유연성, 재사용성, 테스트 용이성을 개선해 준다.

## Item6 : 불필요한 객체 생성을 피하라

```java
String s = new String("naver");
String s = "naver";
//다음 두 코드의 차이는?
//첫 번 째 방식은 인스턴스를 매번 생성하고, 두번째 방식은 같은 인스턴스를 재사용 한다.
Boolean(String);
Boolean.valueOf(String);
//아래 방식이 낫다
```
다음 두 방식을 비교해보자

```java

//방법 1
static boolean isRomanNumeralSlow(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
//방법 2
private static final Pattern ROMAN = Pattern.compile(
        "^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

static boolean isRomanNumeralFast(String s) {
    return ROMAN.matcher(s).matches();
}

```
- String.matches 는 성능이 중요한 상황에서 반복사용하기에 적합하지 않다.

불필요한 오토박싱
* 박싱 : primitive type > wrapper class
* 언박싱 : wrapper class > primitive type

```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum;
}
```

- Long > long 으로하면 불필요한 객체 생성을 줄일 수 있다.

## Item7 : 다 쓴 객체 참조를 해제하라

다음 코드에서 문제가 있을까?
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

- pop을 다음과 같이 수정.
- capacity 늘리는 부분에서 OutOfMemoryError 나타남.
  - 메모리의 끝과 스택의 끝이 일치하지 않기 때문
```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```

- 자기 메모리를 직접 관리하는 클래스라면 메모리 누수에 주의해야 한다.
  - 비활성 영역의 객체가 쓸모없다는 것은 gc 도 모른다.
  - null처리하여 gc에 알려야 한다.
- 캐시가 필요하면 `WeakHashMap`을 사용한다.
  - WeakHashMap에 넣은 값은 키 값이 더이상 사용되지 않을 때 제거된다. 
  - 이 말인 즉, 그 키 값을 가리키는 참조값이 사라졌을 때를 이야기 한다.
  - 예시 <https://www.baeldung.com/java-weakhashmap>

