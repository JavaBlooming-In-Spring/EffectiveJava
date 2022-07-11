## 질문

[item15]에서는 클래스에서 배열 필드는 final이어도 public으로 두면 안된다고 말하고 있습니다.
100p 두 번째 예시를 보면 이를 위한 대안으로
```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
  return PRIVATE_VALUES.clone();
}
```
을 호출하고 있는데, PRIVATE_VALUES 필드가 결국 primitive배열이 아니라면, 내부 요소가 바뀔 위험이 있는 게 아닌가 하는 생각이 듭니다.

## 문제

`clone`을 통해 얻은 배열 참조 원소의 내부의 값을 변경하게 된다면 결국 원본의 배열도 바뀌게 되는 문제가 발생합니다.
```java
public class Thing {

  private String name;

  public Thing(String name) {
    this.name = name;
  }

  public void setName(String name) {
    this.name = name;
  }

  @Override
  public String toString() {
    return "name : " + this.name;
  }
}
```
다음과 같이 `Thing` 클래스가 정의되어 있습니다. 가변 필드를 가지고 있기에 값을 변경할 수 있는 `setter` 메서드가 존재합니다.
```java
public class ThingSafeCopy {

  private static final Thing[] PRIVATE_VALUES = { new Thing("tv"), new Thing("phone") };
  public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
  }

  public Thing[] getThings() {
    return PRIVATE_VALUES;
  }

}
```
다음은 `복사본`을 반환하는 메서드를 가진 클래스를 정의하였습니다. 원본 배열을 반환하는 함수는 존재해서는 안되지만 테스트를 위해 추가하였습니다.
```java
public class ThingMain {

  public static void main(String[] args) {
    Thing[] copyThings = ThingSafeCopy.values();
    Thing[] originalThings = new ThingSafeCopy().getThings();

    System.out.println(copyThings); // Thing;@75bd9247
    System.out.println(originalThings); // Thing;@7d417077

    copyThings[0].setName("book");
    System.out.println(copyThings[0]); // name : book
    System.out.println(originalThings[0]); // name : book
  }

}
```
원본 배열과 복사 배열은 다른 참조를 가지고 있습니다. 하지만 원소의 참조값은 동일하기에 복사 배열의 원소의 값을 조작하게 되면 원본 배열에도 영향을 주게 됩니다.

이는 결국 내부 원소가 바뀔 위험성이 존재함을 의미합니다.

## 해결

가장 간단한 해결책은 원소로 가지는 값을 불변으로 만드는 것입니다.
```java
public class Thing {

  private final String name;

  public Thing(String name) {
    this.name = name;
  }

  @Override
  public String toString() {
    return "name : " + this.name;
  }
}
```
다음과 같이 생성자에 의해 값이 정해지고 값이 바뀔 가능성이 없다면 `clone`을 사용하더라도 원본 배열이 바뀔 가능성이 존재하지 않습니다.

다음 방법으로는 사용하고자 하는 참조에 `clone`을 구현해 깊은 복사가 가능하도록 하는 것입니다.
```java
public class Thing implements Cloneable {

  private String name;

  public Thing(String name) {
    this.name = name;
  }

  public void setName(String name) {
    this.name = name;
  }

  @Override
  public String toString() {
    return "name : " + this.name;
  }

  @Override
  public Thing clone() {
    return new Thing(this.name);
  }
}
```
`Cloneable` 인터페이스를 구현하도록 하여 `clone` 메서드를 재정의합니다.
```java
public class ThingSafeCopy {

  private static final Thing[] PRIVATE_VALUES = { new Thing("tv"), new Thing("phone") };
  public static final Thing[] values() {
    Thing[] things = new Thing[PRIVATE_VALUES.length];
    for (int i = 0; i < PRIVATE_VALUES.length; i++) {
      things[i] = PRIVATE_VALUES[i].clone();
    }
    return things;
  }

  public Thing[] getThings() {
    return PRIVATE_VALUES;
  }

}
```
`Thing`에서 구현한 `clone` 메서드를 활용하여 깊은 복사가 이루어진 배열을 반환하도록 함수를 수정합니다.
```java
public class ThingMain {

  public static void main(String[] args) {
    Thing[] copyThings = ThingSafeCopy.values();
    Thing[] originalThings = new ThingSafeCopy().getThings();

    System.out.println(copyThings); // Thing;@75bd9247
    System.out.println(originalThings); // Thing;@7d417077

    copyThings[0].setName("book");
    System.out.println(copyThings[0]); // name : book
    System.out.println(originalThings[0]); // name : tv
  }

}
```
복사 배열의 원소는 원본 배열의 원소와 다른 참조 값을 가지기에 복사 배열의 원소 내부의 값이 변경되어도 원본 배열에 영향을 주지 않습니다.

## 결론
변경 가능성을 최소화시키기 위해 불변을 적극적으로 활용하는 것이 가장 좋은 방법이나, 예외적으로 구현하기 위해서는 직접 메서드를 정의하여 복사가 이루어지도록 해야 합니다.