> ### 질문
> `p89`쪽에 기존 클래스를 확장한 구체 클래스에서 `compareTo` 규약을 지킬 수 없기에 내부적으로 클래스를 두는 방법을 선택한다고 합니다. 두 방법 모두 확장된 클래스에서 `compareTo`를 구현하게 된다면 규약을 지킬 수 있다는 생각이 들었는데, 이해를 잘못하고 있는 것 같아 정리가 필요할 것 같습니다.

## 클래스를 확장 했을 떄 `Comparable` 문제
아래와 같은 코드가 있다고 가정해보겠습니다.

```java
@AllArgsConstructor
public class Point implements Comparable<Point> {

  public Integer x;
  public Integer y;

  @Override
  public int compareTo(Point o) {
    if (x.compareTo(o.x) == 0) {
      return y.compareTo(o.y);
    }
    return x.compareTo(o.x);
  }
}
```
```java
public class PointColor extends Point implements Comparable<Point> {

  public String color;

  public PointColor(Integer x, Integer y, String color) {
    super(x, y);
    this.color = color;
  }

  @Override
  public int compareTo(Point o) {
    int result = super.compareTo(o);
    if (result == 0) {
      return color.compareTo(((PointColor) o).color);
    }
    return result;
  }
}
```

위와 같이 코드가 있을 때, `compareTo` 규약을 지킬 수 없습니다. 아래 예시를 보겠습니다.
```java
class PointColorTest {

  public static void main(String[] args) {
    Point point = new Point(1, 2);
    PointColor pointColor = new PointColor(1, 2, "RED");

    System.out.println("point.compareTo(pointColor): " + point.compareTo(pointColor) + " == " + 0);

    try {
      System.out.println(pointColor.compareTo(point) + " == " + 0);
    } catch (ClassCastException e) {
      System.out.println("ClassCastException");
    }
  }
}
```

위 코드의 결과는 아래와 같습니다.
```
point.compareTo(pointColor): 0 == 0
ClassCastException
```
위와 같은 현상이 발생하는 이유는, `compareTo`를 재정의하는 과정에서 `ClassCastException`이 발생할 수 있는 코드를 작성하게 되기 때문입니다. 따라서 위 코드는 아래와 같이 수정되어야 규약을 지킬 수 있습니다.

```java
public class GoodPointColor implements Comparable<GoodPointColor> {

  Point point;
  String color;

  @Override
  public int compareTo(GoodPointColor o) {
    int result = point.compareTo(o.point);
    if (result == 0) {
      return color.compareTo(o.color);
    }
    return result;
  }
}
```

위와 같이 `GoodPointColor`처럼 사용하여야 서로 다른 `Class`에 대한 `compareTo` 비교를 컴파일 단계에서 막을 수 있습니다.