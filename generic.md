# 제네릭에 대해 알아보자!

<br /><br />

* generic
```
자바의 제네릭은 데이터 형식을 매개변수화하여 클래스나 메서드에서 사용할 수 있도록 하는 기능이다.
제네릭을 사용하면 클래스나 메서드를 일반적으로 작성하여 재사용성을 높일 수 있다.
이는 유형 안전성을 제공하고 런타임 오류를 방지하는 데 도움이 된다.

1) 타입 안정성(Type Safety) -> 컴파일 시간에 컴파일러가 형식 검사를 수행하여 잘못된 형식의 데이터가 들어가는 것을 방지.
2) 코드 중복 최소화 -> 제네릭을 사용하면 같은 기능을 하는 코드를 여러 번 작성하는 것을 피할 수 있다.
3) 가독성 향상 -> 제네릭을 사용하면 코드가 보다 간결하고 이해하기 쉬워진다.
```

<br /><br /><br />

1. 제네릭에서 사용되는 다른 약자들
```
1) T -> Type (타입)
2) E -> Element (요소)
3) K -> Key (키)
4) V -> Value (값)
5) N -> Number (숫자)
6) S, U, V 등 -> 다른 용도로 사용될 수 있으며, 사용자가 원하는 약자 선택 가능

이러한 약자들은 제네릭 타입 매개변수를 통해 타입의 추상화를 가능하게 하며,
보다 일반적이고 유연한 코드 작성을 도와준다.
```

<br /><br /><br />

2. 예시
```java
/* 제네릭 클래스 */

public class Box<T> {

  private T content; /* 제네릭 필드 */

  public Box(T content) { /* 제네릭 생성자 */
    this.content = content;
  }

  public void setContent(T content) { /* 제네릭 메서드(Setter) */
    this.content = content;
  }

  public T getContent() { /* 제네릭 메서드(Getter) */
    return content;
  }
}

/* 사용 예시 */
Box<String> stringBox = new Box<>("Hello");
String content = stringBox.getContent(); /* "Hello" */
```
```
Box 클래스는 제네릭 타입(T)을 사용하여 구현된다.
이렇게 함으로써 Box 객체를 생성할 때 어떤 타입의 데이터를 저장할 지 결정할 수 있다.

Box 클래스의 생성자는 T 타입의 매개변수를 받는다.
= Box 객체를 생성할 때 데이터를 초기화할 수 있다.

다이아몬드 연산자(<>)를 사용하여 Box 객체를 생성할 때,
실제 타입 파라미터를 명시하지 않고도 컴파일러가 타입을 추론하게 된다.
= new Box<>("Hello")는 컴파일러에 의해 new Box<String>("Hello")로 처리된다.
```
