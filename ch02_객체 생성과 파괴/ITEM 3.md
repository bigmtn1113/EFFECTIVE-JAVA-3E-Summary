`싱글턴(singleton)`이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

그런데 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워 질 수 있다.  
타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 `가짜(mock)` 구현으로 대체할 수 없기 때문이다.

싱글턴 만드는 방식은 보통 둘 중 하나다.
- **public static final 필드 방식의 싱글턴**
- **정적 팩터리 방식의 싱글턴**

## public static final 필드 방식의 싱글턴
```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
}
```

생성자는 필드가 초기화될 때 한 번만 호출된다.  
여기서 만들어진 인스턴스는 전체 시스템에서 하나뿐임이 보장된다.

단, 권한이 있는 클라이언트는 `리플렉션 API`([ITEM 65]())인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다.  
이러한 상황을 막으려면 생성자를 수정해 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.

### 장점
- **해당 클래스가 싱글턴임이 API에 명백히 드러난다.**
- **간결하다.**

<br/>

## 정적 팩터리 방식의 싱글턴
```java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  public static Elvis getInstance() { return INSTANCE; }
}
```

Elvis.getInstance는 항상 같은 객체의 참조를 반환하므로 제2의 Elvis 인스턴스는 만들어지지 않는다.  
(역시 리플렉션을 통한 예외는 똑같이 적용된다.)

### 장점
- **API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.**  
  클라이언트 코드를 고치지 않고 팩터리 메소드 내용을 고쳐서 다른 인스턴스를 넘길 수 있다는 얘기다.  
  팩터리 메소드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있다.
  
- **정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다([ITEM 30]()).**

- **정적 팩터리의 메소드 참조를 공급자(supplier)로 사용할 수 있다.**
  
  ```java
  Supplier<Elvis> elvis = Elvis::getInstance;
  ```
  
  이런 식으로 사용할 수 있다([ITEM 43](), [ITEM 44]()).

-> 이러한 장점들이 필요하지 않다면 public 필드 방식이 좋다.

<br/>

## 직렬화할 때 주의사항
모든 인스턴스 필드를 `일시적(transient)`이라고 선언하고 `readResolve` 메서드를 제공해야 한다([ITEM 89]()).  
이렇게 하지 않으면 역직렬화할 때마다 새로운 인스턴스가 만들어진다.

```java
// 싱글턴임을 보장해주는 readResolve 메서드
private Object readResolve() {
  // 진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
  return INSTANCE;
}
```

### ※ 참고
**직렬화(Serialization)**  
자바 시스템 내부에서 사용되는 `Object` 또는 `Data`를 외부의 자바 시스템에서도 사용할 수 있도록 `byte` 형태로 데이터를 변환하는 기술이다.  
객체를 파일로 저장하거나, 네트워크로 전송할 목적이라면 클래스를 선언할 때 `implements Serializable`을 추가해야 한다.  
이것은 `JVM`에게 직렬화해도 좋다고 승인하는 역할을 한다.

**역직렬화**  
`byte`로 변환된 Data를 원래대로 `Object`나 `Data`로 변환하는 기술이다.

**transient**  
직렬화하는 과정에서 `제외`하고 싶을 때 사용한다.  
보안 상의 이유나 다른 이유로 데이터를 전송하고 싶지 않을 때 사용한다.

<br/>

## 열거 타입 방식의 싱글턴 - 바람직한 방법
```java
public enum Elvis {
  INSTANCE;
}
```

### 장점
- **public 필드보다 간결하다.**
- **추가 노력 없이 직렬화 가능하다.**
- **복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 막아준다.**

단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법을 사용할 수 없다.
