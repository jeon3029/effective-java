# 객체 생성과 파괴

## Item1 : 생성자 대신 팩터리 메서드를 고려하자

- 장점
1. 이름을 가질 수 있음
2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 됨
3. 반환 타입의 하위타입 객체를 반환할 수 있는 능력이 있다
4. 입력 매개변수에 따라 다른 클래스의 객체를 반환할 수 있음
5. 정적 팩터리 메서드를 작성하는 시점에 반환할 객체의 클래스가 없어도 됨

- 단점
1. 상속을 하려면 `public` `protected` 생성자 가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.
2. 프로그래머가 찾기 어렵다

- 예시
```java
Date d = Date.from(instant);
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
StackWalker luke = StackWalker.getInstance(options);
Object newArray = Array.newInstance(classObject, arrayLen);
FileStore fs = Files.getFileStore(path);
BufferedReader br = Files.newBufferedReader(path);
List<Complaint> litany = Collections.list(legacyLitany);
```

상대적인 장단점을 이해하고 사용하자

## Item2 : 생성자에 매개 변수가 많다면 빌더를 고려하자

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
- 객체가 완성되기 전까지 consistency 가 무너진 상태

3. 빌더패턴
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