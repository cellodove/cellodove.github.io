# 챕터6 객체와 자료 구조

![Bridge.jpg](/assets/images/Bridge.jpg?raw=true)

변수를 private으로 정의하는 이유가 있다. 남들이 변수에 의존하지 않았으면 싶어서이다. 그런데 수 많은 프로그래머가 get, set 함수를 당연하게 public해 변수를 노출할까?

## 6.1 자료 추상화

변수를 private로 선언하더라도 각 값마다 get, set 함수를 제공한다면 구현을 외부로 노출하는 셈이다.

```java
// 구체적인 Point 클래스
// 명확하게 직교 좌표계를 쓴다는 것을 알 수 있다.
public class Point {
    public double x;
    public double y;
}

// 추상적인 Point 클래스
// 클래스 메서드가 접근 정책을 강제한다.
public interface Point {
    double getX();
    double getY(); // 조회는 각각 가능하지만
    void setCartesian(double x, double y); // 설정을 2개의 값을 동시에 넣어주어야 한다.
    double getR();
    double getTheta();
    void setPolar(double r, double theta);
```

```java
// 구체적인 Vehicle 클래스
// 변수를 그대로 리턴하는 함수일 것이 틀림없다.
public interface Vehicle {
    public getFuelThankCapacityInGallons();
    public getGallonsOfGasoline();
}

// 추상적인 Vehicle 클래스
// 백분율이라는 추상적인 개념으로 반환하기에 어디서 오는지 사용자에게 드러나지 않는다.
public interface Vehicle {
    double getPercentFuelRemaining();
}
```

**자료를 세세하게 공개하기 보다는 추상적인 개념으로 표현하는 편이 낫다.**

**get, set함수로 변수를 다룬다고 클래스가 되지는 않는다. 그보다는 추상 인터페이스를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스이다.**

## 6.2 자료/객체 비대칭

- 객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다.
- 자료 구조는 자료를 그대로 공개하며 별다른 함수는 제공하지 않는다.

### 절차적인 도형 클래스

```java
public class Square {
    public Point topLeft;
    public double side;
}
```

```java
public class Rectangle {
    public Point topLeft;
    public double height;
    public double width;
}
```

```java
public class Circle {
    public Point center;
    public double radius;
    public double width;
}
```

```java
public class Geometry {
    public final double PI = 3.141592653585793;

    public double area(Object shape) throws NoSuchShapeException {
        if(shape instanceOf Square) {
            Square s = (Square)shape;
            return s.side * s.side;
        }
    else if(shape instanceOf Rectangle) {
            Rectangle r = (Rectangle)shape;
            return r.height * r.width;
        }
    else if(shape instanceOf Circle) {
            Circle c = (Circle)shape;
            return PI * c.radius * c.radius
        }
    }
}
```

Geometry 클래스는 세 가지 도형 클래스를 다룬다. 각 도형 클래스는 간단한 자료 구조다.

아무 메서드도 제공하지 않고, 도형이 동작하는 방식은 Geometry 클래스에서 구현한다.

만약 Geometry 클래스에 함수를 추가하고 싶다면 도형 클래스는 아무 영향도 받지 않는다. 하지만

새 도형이 추가 된다면 Geometry 클래스에 속한 함수를 모두 고쳐야 한다.

### 객체 지향적인 도형 클래스

```java
public class Square implements Shape {
    public Point topLeft;
    public double side;

    public double area() {
        return side*side;
    }
}
```

```java
public class Rectangle implements Shape {
    public Point topLeft;
    public double height;
    public double width;

    public double area() {
        return height*width;
    }
}
```

```java
public class Circle implements Shape {
    public Point center;
    public double radius;
    public double width;

    public double area() {
        return PI*radius*radius;
    }
}
```

area() 는 다형 메서드다. Geometry 클래스는 필요 없다.

새 도형을 추가해도 기존 함수에 아무런 영향을 미치지 않는다. 하지만 새 함수를 추가하고 싶다면 도형 클래스를 전부 고쳐야 한다.

- 자료 구조를 사용하는 절차적인 코드는 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다. 반면, 객체 지향 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다.
- 절차적인 코드는 새로운 자료 구조를 추가하기 어렵다. 그러려면 모든 함수를 고쳐야 한다. 객체 지향 코드는 새로운 함수를 추가하기 어렵다. 그러려면 모든 클래스를 고쳐야 한다.

**객체 지향 코드에서 어려운 변경은 절차적인 코드에서 쉬우며, 반대는 객체 지향 코드에서 쉽다!**

## 6.3 디미터 법칙

디미터 법칙은 잘 알려진 휴리스틱heuristic(경험에 기반하여 문제를 해결하거나 학습하거나 발견해 내는 방법)으로, **모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙이다.**

객체는 조회 함수로 내부 구조를 공개하면 안된다. 하지만 자료구조는 당연히 내부 구조를 드러내야한다. 그리고 자료구조가 빈이라면 표준 프레임워크의 정의를 따라 set() 메소드와 get() 메소드를 구현해야한다.

좀 더 정확히 표현하자면, 디미터 법칙은 "클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다"고 주장한다.

- 클래스 C
- f가 생성한 객체
- f 인수로 넘어온 객체
- C 인스턴스 변수에 저장된 객체

하지만 위 객체에서 허용된 메서드가 반환하는 객체의 메서드는 호출하면 안 된다. 다시 말해,**낯선 사람은 경계하고 친구랑만 놀라는 의미**이다.

### 기차 충돌

다음과 같은 코드를 기차 충돌train wreck이라 부른다.

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

여러 대의 객차가 한 줄로 이어진 기차처럼 보이기 때문이다. 일반적으로 조잡하다 여겨지는 방식이므로 피하는 편이 좋다.

위 코드는 다음과 같이 나누는 편이 낫다.

```java
Options opts = ctxt.getOptions(); 
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

위 예제가 디미터 법칙을 위반하는지 여부는 위의 변수들이 객체인지 자료 구조인지에 달렸다. 객체라면 내부 구조를 숨겨야 하므로 확실히 디미터 법칙을 위반한다. 반면, 자료 구조라면 당연히 내부 구조를 노출하므로 문제되지 않는다.

### 잡종 구조

이런 혼란으로 객체와 자료구조가 뒤섞인 잡종 구조가 나온다.

이런 잡종 구조는 새로운 함수는 물론이고 새로운 자료구조도 추가하기 어렵다. 그러므로 잡종 구조는 되도록 피하도록 한다. **프로그래머가 함수나 타입을 보호할지 공개할지 확신하지 못해 (더 나쁘게는 무지해) 어중간하게 내놓은 설계에 불과하다.**

### 구조체 감추기

객체 메소드 내에서 자신이 몰라야 할 객체를 알 필요는 없다. 즉 몰라도 문제 없는 객체를 굳이 속을 드러내도록 하지않는다.

그러므로 어떤 객체를 드러내라고 시키기 보단 어떤 행동을 하도록 만든다.

## 6.4 자료 전달 객체

자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스다. 이를 때로는 자료 전달 객체Data Transfer Object, DTO라 한다.

- 데이터베이스와 통신하거나 소켓에서 받은 메시지의 구문을 분석할 때 유용
- 가공되지 않은 정보를 애플리케이션 코드에서 사용할 객체로 변환하는 일련의 단계에서 처음 사용되는 구조체

```java
class Address {
    private final String postalCode;
    private final String city;
    private final String street;
    private final String streetNumber;
    private final String apartmentNumber;

    public Address(String postalCode, String city, String street, String streetNumber, String apartmentNumber) {
        this.postalCode = postalCode;
        this.city = city;
        this.street = street;
        this.streetNumber = streetNumber;
        this.apartmentNumber = apartmentNumber;
    }

    public String getPostalCode() {
        return postalCode;
    }

    public String getCity() {
        return city;
    }

    public String street() {
        return street;
    }

    public String streetNumber() {
        return streetNumber;
    }

    public String apartmentNumber() {
        return apartmentNumber;
    }
}
```

### 활성 레코드

*활성 레코드*는 DTO의 특수한 형태입니다.

- 공개 변수가 있거나 비공개 변수에 조회/설정 함수가 있는 자료구조
- save나 find와 같은 탐색 함수도 제공

활성 레코드에 비즈니스 규칙 메서드를 추가해 객체로 취급하는 경우가 있는데 바람직하지 않다.

잡종 구조가 나오기 때문이다. **활성 레코드는 자료 구조로 취급한다.**

비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성한다.(내부 자료는 활성 레코드의 인스턴스일 가능성이 높다.)

## 6.5 결론

- 객체는 동작을 공개하고 자료를 숨긴다. 그래서 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉬운 반면. 기존 객체에 새 동작을 추가하기는 어렵다.
- 자료 구조는 별다른 동작 없이 자료를 노출한다. 그래서 기존 자료 구조에 새 함수를 추가하기는 쉬우나 기존 함수에 새 자료 구조를 추가하기는 어렵다.

**시스템을 구현할 때, 새로운 자료 타입을 추가하는 유연성이 필요하다. 그렇다면 그 부분은 객체가 더 적합하다.**  

**어떤 때는 새로운 동작을 추가하는 유연성이 필요하다. 그렇다면 그 부분은 자료 타입과 절차적 코드가 더 적합하다.**

## 참조

> 로버트 C. 마틴, 『Clean Code 클린 코드』, 박재호, 이해영 옮김, 케이앤피북스(2010), P148-158.
>