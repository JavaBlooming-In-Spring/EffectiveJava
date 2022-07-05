## 질문
빌더 패턴을 공부하던 도중 `Builder`라는 내부 클래스를 사용하게 되는데 내부 클래스의 `private field` 접근이 가능한 것을 확인했습니다.

```java
public class BuilderPattern {

  private int param01;
  private int param02;
  private int param03;
  private int param04;
  private int param05;

  private BuilderPattern(Builder builder) {
    this.param01 = builder.param01;
    this.param02 = builder.param02;
    this.param03 = builder.param03;
    this.param04 = builder.param04;
    this.param05 = builder.param05;
  }

  // 중략

  public static class Builder {
    private int param01;
    private int param02;
    private int param03;
    private int param04;
    private int param05;

    // 중략
  }
}
```

이러한 접근이 왜 가능한건지와 내부 클래스 생성 시 메모리에는 어떻게 올라가게 되는지 궁금합니다.

## private 접근 제어자의 범위
`class` 내부적으로 같은 네임스페이스라면 접근 가능합니다. 따라서 내부 클래스 안에서는 서로 참조가 가능합니다.
<img width="745" alt="image" src="https://user-images.githubusercontent.com/26597702/177268499-1eb71a06-a321-42b4-b66a-367cc207772f.png">

## Inner Class / Outer Class
해당 질문에 대한 답변 외에 `java`의 `nested class`에 대해 어떻게 동작하는 지 알아보았습니다.

```java
public class OuterClass {

  private int outerField;

  public static class PublicStaticInnerClass {

    private int innerField;

    public PublicStaticInnerClass(int _outerField) {
      outerField = _outerField; // static필드가 아닌 필드에 접근 에러!
    }
  }

  private static class PrivateStaticInnerClass {

    private int innerField;

    public PrivateStaticInnerClass(int _outerField) {
      outerField = _outerField; // static필드가 아닌 필드에 접근 에러!
    }
  }

  public class PublicInnerClass {

    private int innerField;

    public PublicInnerClass(int outerField) {
      OuterClass.this.outerField = outerField; // 이름이 겹치므로 정확하게 명시해 줘야 함
    }
  }

  private class PrivateInnerClass {

    private int innerField;

    public PrivateInnerClass(int outerField) {
      OuterClass.this.outerField = outerField; // 이름이 겹치므로 정확하게 명시해 줘야 함
    }
  }
}
```

`static` 필드를 제외하곤 `InnerClass`에서 `OuterClass`의 필드에 접근 가능했습니다. **다만 name이 겹칠 경우 정확하게 어떤 곳의 필드인지 명시해줘야 합니다.**

```java
public class OuterClass {

  private int outerField;

  public OuterClass(
      PublicStaticInnerClass publicStaticInnerClass,
      PrivateStaticInnerClass privateStaticInnerClass,
      PublicInnerClass publicInnerClass,
      PrivateInnerClass privateInnerClass) {
    this.outerField = privateInnerClass.innerField;
    this.outerField = privateStaticInnerClass.innerField;
    this.outerField = publicInnerClass.innerField;
    this.outerField = privateInnerClass.innerField;
  }
}
```

`Outer Class`에선 `Inner class`의 필드가 `private`으로 되어 있더라도 모두 접근 가능합니다.