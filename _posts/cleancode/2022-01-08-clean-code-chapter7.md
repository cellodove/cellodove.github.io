---
title : "[클린 코드] 챕터7 오류처리"
excerpt: "클린코드 정주행10"

categories :
  - CleanCode

toc: true
toc_sticky: true
last_modified_at: 2022-01-08
---


![chapter8_error.png](/assets/images/chapter8_error.png?raw=true)

오류처리는 프로그램에 반드시 필요한 요소 중 하나다. 뭔가 잘못될 가능성은 늘 존재한다.

오류 처리는 중요하다. 하지만 오류 처리 코드로 인해 프로그램 논리를 이해하기 어려워진다면 클린 코드라 부르기 어렵다.

이 장에서는 깨끗하고 튼튼한 코드에 한 걸음 더 다가가는 단계로 우아하고 고상하게 오류를 처리하는 기법과 고려 사항 몇 가지를 소개한다.

## 7.1 오류 코드보다 예외를 사용하라

```java
public class DeviceController {
    ...
    public void sendShutDown() {
        DeviceHandle handle = getHandle(DEV1);
        // 디바이스 상태를 점검한댜.
        if (handle != DeviceHandle.INVALID) {
            // 레코드 필드에 디바이스 상태를 저장한다.
            retrieveDeviceRecord(handle);
            // 디바이스가 일시정지 상태가 아니라면 종료한다.
            if (record.getStatus() != DEVICE_SUSPENDED) {
                pauseDevice(handle);
                clearDeviceWorkQueue(handle);
                closeDevice(handle);
            } else {
                logger.log("Device suspended. Unable to shut down");
            }
        } else {
            logger.log("Invalid handle for: " + DEV1.toString());
        }
    }
    ...
}
```

오류 코드 사용시 문제점

- 함수를 호출한 즉시 오류를 확인해야 하기 때문에 호출자 코드가 복잡해진다.
- 오류 확인을 잊어버리기 쉽다.

```java
public class DeviceController {
    ...
    public void sendShutDown() {
        try {
            tryToShutDown();
        } catch (DeviceShutDownError e) {
            logger.log(e);
        }
    }

    private void tryToShutDown() throws DeviceShutDownError {
        DeviceHandle handle = getHandle(DEV1);
        DeviceRecord record = retrieveDeviceRecord(handle);
        pauseDevice(handle); 
        clearDeviceWorkQueue(handle); 
        closeDevice(handle);
    }

    private DeviceHandle getHandle(DeviceID id) {
        ...
        throw new DeviceShutDownError("Invalid handle for: " + id.toString());
        ...
    }
    ...
}
```

예외를 던지는 방법은 코드를 깔끔하게 만들어 논리가 뒤섞이지 않게 해준다. 조건문이 없어짐으로 메서드를 수행하는 코드들이 한 눈에 보일 수 있게 된다.

- 가독성 향상
- 기능 알고리즘과 오류 처리 알고리즘의 분리

## 7.2 Try-Catch-Finally 문부터 작성하라

예외에서 프로그램 안에다 범위를 정의한다는 사실은 매우 흥미롭다.

try 블록은 어떤 면에서 트랜잭션과 비슷하다. try 블록에서 무슨 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지하기 때문이다.

그러므로 코드를 짤때는`try-catch-finally` 문으로 시작하는 편이 낫다. 그러면 try 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다. 즉, 예외 처리를 지정함으로써 프로그래머는 코드의 방향성을 생각하면서 코드를 짤 수 있다.

## 7.3 미확인 예외를 사용하라

안정적인 소프트웨어를 제작하는 요소로 확인된 예외가 반드시 필요하지 않다는 사실이 분명해 졌다.

|  | Checked Exception | UnChecked Exception |
| --- | --- | --- |
| 확인 시점 | 컴파일 시점 | 런타임 시점 |
| 처리 여부 | 반드시 처리 | 명시적으로 처리하지 않아도 됨 |
| 트랜잭션 처리 | roll-back 하지 않음 | roll-back 함 |
| 예 | IOException, ClassNotFoundException | NullPointerException, ArithmeticException |

최상위 함수가 아래 함수를 호출한다. 아래 함수는 그 아래 함수를 호출한다. 단계를 내려갈수록 호출하는 함수 수는 늘어난다.

최하위 함수를 변경해 새로운 오류를 던진다고 가정하자 확인된 오류를 던진다면 함수는 선언부에 throws절을 추가해야 한다. 그러면 해당 함수를 호출하는 모든 함수가 1) catch 블록에서 새로운 예외를 처리하거나 2) 선언부에 throws절을 추가해야 한다.

결과적으로 최하위 단계에서 최상위 단계까지 연쇄적인 수정이 일어난다!throws경로에 위치하는 모든 함수가 최하위 함수에서 던지는 예외를 알아야 하므로 캡슐화도 깨진다.

## 7.4 예외에 의미를 제공하라

예외를 던질 때는 전후 상황을 충분히 덧붙인다. 그러면 오류가 발생한 원인과 위치를 찾기쉬워진다. 자바는 모든 예외에 호출 스택을 제공한다. 하지만 실패한 코드의 의도를 파악하려면 호출 스택만으로는 부족하다.

오류 메시지에 정보를 담아 예외와 함께 던진다. 실패한 연산 이름과 실패 유형도 언급한다. 응용 프로그램이 로깅 기능을 사용한다면 catch 블록에서 오류를 기록하도록 충분한 정보를 넘겨준다.

## 7.5 호출자를 고려해 예외 클래스를 정의하라

**오류를 정의해 분류하는 방법은 오류를 잡아내는 방법이 되어야 한다.**

- 잘 못된 예 - 외부 라이브러리가 던질 예외를 모두 잡아낸다.

대다수 상황에서 우리가 오류를 처리하는 방식이다.

```java
ACMEPort port = new ACMEPort(12);

try{
    port.open();
} catch (DeviceResponseException e) {
    reportPortError(e);
} catch (ATM1212UnlockedException e) {
    reportPortError(e);
} catch (GMXError e) {
    reportPortError(e);
} finally {
    ...
}
```

- 좋은 예 - 호출하는 라이브러리의 API를 감싸면서 예외 유형을 하나 반환한다.

```java
LocalPort port = new LocalPort(12);
try {
  port.open();
} catch (PortDeviceFailure e) {
  reportError(e);
  logger.log(e.getMessage(), e);
} finally {
  ...
}
```

```java
public class LocalPort {
  private ACMEPort innerPort;
  
  public LocalPort(int portNumber) {
    innerPort = new ACMEPort(portNumber);
  }
  
  public void open() {
    try{
      innerPort.open();
    } catch (DeviceResponseException e) {
      throw new PortDeviceFailure(e);
    } catch (ATM1212UnlockedException e) {
      throw new PortDeviceFailure(e);
    } catch (GMXError e) {
      throw new PortDeviceFailure(e);
    }
  }
  
  ...
}
```

`LocalPort`는 ACMEPort 클래스가 던지는 예외를 잡아 변환하는 Wrapper 클래스이다.

- 좋은 이유

1. 외부 API를 감싸면 외부 라이브러리와 프로그램 사이에 의존성이 크게 줄어든다.
2. 나중에 다른 라이브러리로 갈아타도 프로그램을 뜯어고칠 필요가 줄어든다.
3. Wrapper 클래스에서 외부 API를 호출하는 대신 테스트 코드를 넣어주면 테스트 하기도 쉽다.
4. 특정 업체가 API를 설계한 방식에 국한되지 않는다. 프로그램이 사용하기 편리한 API를 정의할 수 있다.

## 7.6 정상 흐름을 정의하라

외부 API를 감싸 독자적인 예외를 던지고, 코드 위에 처리기를 정의해 중단된 계산을 처리한다. 대개는 멋진 처리 방식이지만, 때로는 적합하지 않을 때도있다.

특수 케이스 패턴으로 클래스를 만들거나 객체를 조작해 특수 케이스를 처리한다.

그러면 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다.

- 특수 케이스 처리 전

```java
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
    m_total += getMealPerDiem();
}
```

- 특수 케이스 처리 후 - 클래스나 객체가 예외적인 상황을 캡슐화 해서 처리한다

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```

`ExpenseReportDAO`를 고쳐 언제나 MealExpense 객체를 반환하게 한다.

청구한 식비가 없다면 일일 기본 식비를 반환하는 MealExpense 객체를 반환한다.

```java
public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
        // 기본값으로 일일 기본 식비를 반환한다.
    }
}
```

## 7.7 null을 반환하지 마라

**null을 반환하고 이를 `if(object != null)`으로 확인하는 방식은 나쁘다.**

- null 대신 예외를 던지거나 특수 사례 객체(ex. `Collections.emptyList()`)를 반환하라
- 사용하려는 외부 API가 null을 반환한다면 Wrapper 를 구현해 예외를 던지거나 특수 사례 객체를 반환하라

나쁜 예

```java
List<Employee> employees = getEmployees();
if (employees != null) {
    for(Employee e : employees) {
        totalPay += e.getPay();
    }
}
```

좋은 예 - *`getEmployees`가 null 대신 빈 리스트를 반환한다.*

```java
List<Employee> employees = getEmployees();
for(Employee e : employees) {
    totalPay += e.getPay();
}
```

**자바의 `Collections.emptyList()`**

```java
public List<Employee> getEmployees() {
    if ( ...직원이 없다면... ) {
        return Collections.emptyList();
    }
}
```

이렇게 코드를 변경하면 코드가 깔끔해질 뿐 아니라 NullPointerException이 발생할 가능성도 줄어든다.

## 7.7 null을 전달하지 마라

메소드에서 null을 반환하는 방식도 나쁘지만 메소드로 null을 전달하는 방식은 더 나쁘다.

NullPointerException이 발생하지 않기위해  예외를 던지거나 assert문을 사용하는 방법도 있다.

하지만 애초에 null을 넘기지 못하도록 금지하는 정책이 합리적이다.

인수로 null이 넘어오면 코드에 문제가 있다는 소리다.

## 7.9 결론

클린 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다. 두 가지는 상충하는 목표가 아니다.

오류 처리를 프로그램 논리와 분리해 독자적인 사안으로 고려하면 튼튼한 클린 코드를 작성 할 수 있다. 오류 처리를 프로그램 논리와 분리하는 정도에 따라서 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아 진다.

## 참조

> 로버트 C. 마틴, 『Clean Code 클린 코드』, 박재호, 이해영 옮김, 케이앤피북스(2010), P160-171.
>