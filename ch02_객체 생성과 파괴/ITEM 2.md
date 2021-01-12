정적 팩터리와 생성자에는 똑같은 제약이 하나 있다. 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 것이다.  
프로그래머들은 이럴 때 `점층적 생성자 패턴(telescoping constructor pattern)`을 즐겨 사용했다.

## 점층적 생성자 패턴(telescoping constructor pattern)
선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식이다.

```java
public class NutritionFacts {
    private final int servingSize;  // (mL, 1회 제공량)     필수
    private final int servings;     // (회, 총 n회 제공량)  필수
    private final int calories;     // (1회 제공량당)       선택
    private final int fat;          // (g/1회 제공량)       선택
    private final int sodium;       // (mg/1회 제공량)      선택
    private final int carbohydrate; // (g/1회 제공량)       선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }    
}
```
```java
NutritionFacts cocaCola =
        new NutritionFacts(240, 8, 100, 0, 35, 27);
```

### 점층적 생성자 패턴의 특징
- 확장하기 어렵다.
- 각 값의 의미가 무엇인지 헷갈릴 것이다.
- 매개변수가 몇 개인지 주의해서 세어 보아야 한다.
- 매개변수 순서를 바꿔도 컴파일러는 알아차리지 못하고 런타임에 엉뚱한 동작을 하게 된다([ITEM 51]()).

-> 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.

<br/>

## 자바빈즈 패턴(JavaBeans pattern)
매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식이다.

```java
public class NutritionFacts {
    // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
    private int servingSize  = -1; // 필수; 기본값 없음
    private int servings     = -1; // 필수; 기본값 없음
    private int calories     = 0;
    private int fat          = 0;
    private int sodium       = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)     { servings = val; }
    public void setCalories(int val)     { calories = val; }
    public void setFat(int val)          { fat = val; }
    public void setSodium(int val)       { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
}
```
```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

### 자바빈즈 패턴의 특징
- 객체 하나를 만드려면 메서드를 여러 개 호출해야 한다.
- 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다.
- 클래스를 불변([ITEM 17]())으로 만들 수 없다.
- 스레드 안전성을 얻으려면 프로그래머가 추가 작업을 해줘야 한다.

<br/>

## 빌더 패턴(Builder pattern)
점층적 생성자 패턴의 `안정성`과 자바빈즈 패턴의 `가독성`을 겸비한 패턴이다.

```java
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
```
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
        .calories(100).sodium(35).carbohydrate(27).build();
```

### 빌더 패턴의 특징
- 클래스가 불변이다.
- 빌더의 세터 메서드들은 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다. `플루언트 API` 혹은 `메서드 연쇄`라 한다.
- 점층적 생성자 패턴보다 읽고 쓰기가 훨씬 간결하고, 자바빈즈 패턴보다 훨씬 안전하다.
- 계층적으로 설계된 클래스와 함께 쓰기에 좋다.
- 생성 비용이 크진 않지만 성능에 민감한 상황에서는 문제가 될 수 있다.
- 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다.

### 계층적으로 설계된 클래스와 잘 어울리는 빌더 패턴
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
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // 아이템 50 참조
    }
}
```
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
}
```
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
}
```
```java
NyPizza pizza = new NyPizza.Builder(SMALL)
        .addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder()
        .addTopping(HAM).sauceInside().build();
```

### ※ 참고
**불변**  
어떠한 변경도 허용하지 않는다는 뜻이다. 대표적으로 String 객체가 있다.

**불변식**  
프로그램이 실행되는 동안, 혹은 정해진 기간 종안 반드시 만족해야 하는 조건을 말한다.  
ex) 리스트의 크기는 반드시 0 이상이어야 한다. 음수 값이 오면 불변식이 깨진 것이다.
