---
title : "[클린 코드] 챕터10 클래스"
excerpt: "클린코드 정주행13"

categories :
  - CleanCode

toc: true
toc_sticky: true
last_modified_at: 2022-01-20
---

![class.jpg](/assets/images/class.jpg?raw=true)

코드의 표현력과 그 코드로 이루어진 함수에 아무리 신경 쓸지라도 좀 더 차원 높은 단계까지 신경 쓰지 않으면 깨끗한 코드를 얻기 어렵다.

이 장에서는 깨끗한 클래스를 다룬다.

## 클래스 체계

클래스를 정의하는 표준 자바 관례에 따르면 클래스 안에서의 순서는 다음과 같다.

1. static public 상수
2. static privte 상수
3. private instance 변수(public 변수가 필요한 경우는 거의 없음)
4. public method
5. private method는 자신을 호출하는 public method 직후에 위치

추상화 단계가 순차적으로 내려간다. 그래서 프로그램은 신문 기사처럼 읽힌다.

### 캡슐화

변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 반드시 숨겨야 하는 것은 아니다.

우리에게 테스트는 중요하므로 테스트를 위해 protected로 선언해서 접근을 허용하기도 한다.

**하지만 비공개 상태를 유지할 온갖 방법을 강구하고, 캡슐화를 풀어주는 결정은 언제나 최후의 수단이다.**

## 클래스는 작아야 한다

클래스를 만들 때 첫 번째 규칙은 크기다. 클래스는 작아야 한다. 두 번째 규칙도 크기다.

클래스를 설계할 때도, 함수와 마찬가지로, ‘작게’가 기본 규칙이라는 의미다.

얼마나 작아야 할까? 클래스가 맡은 책임을 크기로 센다.

```java
public class SuperDashboard extends JFrame implements MetaDataUser {
    public String getCustomizerLanguagePath()
    public void setSystemConfigPath(String systemConfigPath) 
    public String getSystemConfigDocument()
    public void setSystemConfigDocument(String systemConfigDocument) 
    public boolean getGuruState()
    public boolean getNoviceState()
    public boolean getOpenSourceState()
    public void showObject(MetaObject object) 
    public void showProgress(String s)
    public boolean isMetadataDirty()
    public void setIsMetadataDirty(boolean isMetadataDirty)
    public Component getLastFocusedComponent()
    public void setLastFocused(Component lastFocused)
    public void setMouseSelectState(boolean isMouseSelected) 
    public boolean isMouseSelected()
    public LanguageManager getLanguageManager()
    public Project getProject()
    public Project getFirstProject()
    public Project getLastProject()
    public String getNewProjectName()
    public void setComponentSizes(Dimension dim)
    public String getCurrentDir()
    public void setCurrentDir(String newDir)
    public void updateStatus(int dotPos, int markPos)
    public Class[] getDataBaseClasses()
    public MetadataFeeder getMetadataFeeder()
    public void addProject(Project project)
    public boolean setCurrentProject(Project project)
    public boolean removeProject(Project project)
    public MetaProjectHeader getProgramMetadata()
    public void resetDashboard()
    public Project loadProject(String fileName, String projectName)
    public void setCanSaveMetadata(boolean canSave)
    public MetaObject getSelectedObject()
    public void deselectObjects()
    public void setProject(Project project)
    public void editorAction(String actionName, ActionEvent event) 
    public void setMode(int mode)
    public FileManager getFileManager()
    public void setFileManager(FileManager fileManager)
    public ConfigManager getConfigManager()
    public void setConfigManager(ConfigManager configManager) 
    public ClassLoader getClassLoader()
    public void setClassLoader(ClassLoader classLoader)
    public Properties getProps()
    public String getUserHome()
    public String getBaseDir()
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumber()
    public MetaObject pasting(MetaObject target, MetaObject pasted, MetaProject project)
    public void processMenuItems(MetaObject metaObject)
    public void processMenuSeparators(MetaObject metaObject) 
    public void processTabPages(MetaObject metaObject)
    public void processPlacement(MetaObject object)
    public void processCreateLayout(MetaObject object)
    public void updateDisplayLayer(MetaObject object, int layerIndex) 
    public void propertyEditedRepaint(MetaObject object)
    public void processDeleteObject(MetaObject object)
    public boolean getAttachedToDesigner()
    public void processProjectChangedState(boolean hasProjectChanged) 
    public void processObjectNameChanged(MetaObject object)
    public void runProject()
    public void setAçowDragging(boolean allowDragging) 
    public boolean allowDragging()
    public boolean isCustomizing()
    public void setTitle(String title)
    public IdeMenuBar getIdeMenuBar()
    public void showHelper(MetaObject metaObject, String propertyName) 
    // ... many non-public methods follow ...
}
```

예를 들어, SuperDashboard라는 클래스는 공게 메서드가 대략 70개 정도 되는 엄청 큰 만능 클래스이다.

```java
public class SuperDashboard extends JFrame implements MetaDataUser {
    public Component getLastFocusedComponent()
    public void setLastFocused(Component lastFocused)
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumber() 
}
```

하지만 만약 이런 SuperDashboard가 다섯 개 정도의 메서드만 가지고 있다면 어떨까? 만약 메서드 수가 작음에도 불구하고 책임이 너무 많다면 좋지 않다.

클래스 이름은 해당 클래스 책임을 기술해야 한다. 실제로 작명은 클래스 크기를 줄이는 첫번째 관문이다.

간결한 이름이 떠오르지 않는다면 필경 클래스 크기가 너무 커서 그렇다.

클래스 이름이 모호하다면 필경 클래스 책임이 너무 많아서다.

예를 들어, 클래스 이름에 Processor, Manager, Super등과 같이 모호한 단어가 있다면 클래스에다 여러 책임을 떠안았다는 증거다.

또한, 클래스 설명은 만일(”if”), 그리고(”and”), (하)며(”or”), 하지만(”but”)을 사용하지 않고서 25단어 내외로 가능해야 한다.

### 단일 책임 원칙

단일 책임 원칙(Single Responsibility Principle)은 클래스나 모듈을 변경할 이유가 하나, 단 하나뿐이어야 한다는 원칙이다.

SRP는 ‘책임’이라는 개념을 정의하며 적절한 클래스 크기를 제시한다.

클래스는 책임, 즉 변결할 이유가 하나여야 한다는 의미다.

책임, 즉 변경할 이유를 파악하려 애쓰다 보면 코드를 추상화하기도 쉬워진다. 더 좋은 추상화가 더 쉽게 떠오른다.

위에서 예시로 들었던 SuperDashboard에서 버전 정보를 다루는 메서드 세 개를 다로 빼내 Version이라는 독자적인 클래스를 만들면 다른 어플리케이션에서 재사용하기 아주 쉬워진다.

```java
public class Version {
    public int getMajorVersionNumber() 
    public int getMinorVersionNumber() 
    public int getBuildNumber()
}
```

SRP는 객체 지향 설계에서 더욱 중요한 개념이다. 또한 이애하고 지키기 수월한 개념이기도 하다.

하지만 SRP는 클래스 설계자가 가장 무시하는 규칙 중 하나다.

소프트웨어를 돌아가게 만드는 활동과 소프트웨어를 깨끗하게 만드는 활동은 완전히 별개다.

우리들 대다수는 ‘깨끗하고 체계적인 소프트웨어’보다 ‘돌아가는 소프트웨어’에 초점을 맞춘다.

우리는 대부분 프로그램이 돌아가면 일이 끝났다고 여기지만, 이에 더해 만능 클래스를 단일 책임 클래스 여럿으로 분리하는 작업을 잊는다.

그리고 많은 개발자는 자잘한 단일 책임 클래스가 많아지면 큰 그림을 이해하기 어려워진다고 우려한다. 하지만 작은 클래스가 많은 시스템이든 큰 클래스가 몇 개뿐인 시스템이든 돌아가는 부품은 그 수가 비슷하다. 어느 시스템이든 익힐 내용은 그 양이 비슷ㅎ다.

그러므로 고민할 질문은 다음과 같다.

“도구 상자를 어떻게 관리하고 싶은가? 작은 서랍을 많이 두고 기능과 이름이 명확한 컴포넌트를 나눠 넣고 싶은가? 아니면 큰 서랍 몇 개를 두고 모두를 던져 넣고 싶은가?”

규모가 어느 수준에 이르는 시스템은 노리가 많고도 복잡하다. 이런 복잡성을 다루려면 체계적인 정리가 필수다. 그래야 개발자가 무엇이 어디에 있는지 쉽게 찾는다. 그래야 변경을 가할 때 직접 영향이 미치는 컴포넌트만 이해해도 충분하다.

### 응집도

- 클래스는 인스턴스 변수 수가 작아야한다.
- 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 한다.
- 일반적으로 메서드가 변수를 더 많이 사용할수록 메서드와 클래스는 응집도가 더 높다.
- 모든 인스턴스 변수를 메서드마다 사용하는 클래스는 응집도가 가장 높다.

일반적으로 이처럼 응집도가 가장 높은 클래스는 가능하지도 바람직하지도 않다. 그렇지만 우리는 응집도가 높은 클래스를 선호한다.

응집도가 높다는 말은 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미기 때문이다.

```java
// Stack을 구현한 코드, 응집도가 높다.

public class Stack {
    private int topOfStack = 0;
    List<Integer> elements = new LinkedList<Integer>();

    public int size() { 
        return topOfStack;
    }

    public void push(int element) { 
        topOfStack++; 
        elements.add(element);
    }

    public int pop() throws PoppedWhenEmpty { 
        if (topOfStack == 0)
            throw new PoppedWhenEmpty();
        int element = elements.get(--topOfStack); 
        elements.remove(topOfStack);
        return element;
    }
}
```

topOfStack과 element 변수를 각 함수에서 사용하고 있다.

'함수를 작게, 매개변수 목록을 짧게'라는 전략을 따르다 보면 때때로 몇몇 메서드만이 사용하는 인스턴스 변수가 아주 많아진다.

이는 십중팔구 새로운 클래스로 쪼개야 한다는 신호다. 응집도가 낮아지도록 변수와 메서드를 적절히 분리해서 새로운 클래스 두 세개로 쪼개준다.

### 응집도를 유지하면 작은 클래스 여럿이 나온다

큰 함수를 작은 함수 여럿으로 나누기만 해도 클래스 수가 많아진다. 예를 들어, 변수가 아주 많은 큰 함수가 하나 있다

- 큰 함수 일부를 작은 함수로 빼내고 싶다
- 빼내려는 코드가 큰 함수에 정의 된 변수를 많이 사용한다
- 변수들을 새 함수에 인수로 넘겨야 하나? 아니다.
- 변수들을 클래스 인스턴스 변수로 승격 시키면 인수가 필요없다. 하지만 응집력이 낮아진다.
- 몇몇 함수가 몇몇 인스턴스 변수만 사용한다면 독자적인 클래스로 분리해도 된다

```java
package literatePrimes;

public class PrintPrimes {
    public static void main(String[] args) {
        final int M = 1000; 
        final int RR = 50;
        final int CC = 4;
        final int WW = 10;
        final int ORDMAX = 30; 
        int P[] = new int[M + 1]; 
        int PAGENUMBER;
        int PAGEOFFSET; 
        int ROWOFFSET; 
        int C;
        int J;
        int K;
        boolean JPRIME;
        int ORD;
        int SQUARE;
        int N;
        int MULT[] = new int[ORDMAX + 1];

        J = 1;
        K = 1; 
        P[1] = 2; 
        ORD = 2; 
        SQUARE = 9;

        while (K < M) { 
            do {
                J = J + 2;
                if (J == SQUARE) {
                    ORD = ORD + 1;
                    SQUARE = P[ORD] * P[ORD]; 
                    MULT[ORD - 1] = J;
                }
                N = 2;
                JPRIME = true;
                while (N < ORD && JPRIME) {
                    while (MULT[N] < J)
                        MULT[N] = MULT[N] + P[N] + P[N];
                    if (MULT[N] == J) 
                        JPRIME = false;
                    N = N + 1; 
                }
            } while (!JPRIME); 
            K = K + 1;
            P[K] = J;
        } 
        {
            PAGENUMBER = 1; 
            PAGEOFFSET = 1;
            while (PAGEOFFSET <= M) {
                System.out.println("The First " + M + " Prime Numbers --- Page " + PAGENUMBER);
                System.out.println("");
                for (ROWOFFSET = PAGEOFFSET; ROWOFFSET < PAGEOFFSET + RR; ROWOFFSET++) {
                    for (C = 0; C < CC;C++)
                        if (ROWOFFSET + C * RR <= M)
                            System.out.format("%10d", P[ROWOFFSET + C * RR]); 
                    System.out.println("");
                }
                System.out.println("\f"); PAGENUMBER = PAGENUMBER + 1; PAGEOFFSET = PAGEOFFSET + RR * CC;
            }
        }
    }
}
```

들여쓰기가 심하고, 이상한 구조가 많고, 구조가 빽빽하게 결합되었다. 최소한 여러 함수로 나눠야 마땅하다.

리팩토링을 해보자.

```java
package literatePrimes;

public class PrimePrinter {
    public static void main(String[] args) {
        final int NUMBER_OF_PRIMES = 1000;
        int[] primes = PrimeGenerator.generate(NUMBER_OF_PRIMES);

        final int ROWS_PER_PAGE = 50; 
        final int COLUMNS_PER_PAGE = 4; 
        RowColumnPagePrinter tablePrinter = 
            new RowColumnPagePrinter(ROWS_PER_PAGE, 
                        COLUMNS_PER_PAGE, 
                        "The First " + NUMBER_OF_PRIMES + " Prime Numbers");
        tablePrinter.print(primes); 
    }
}
```

```java
package literatePrimes;

import java.io.PrintStream;

public class RowColumnPagePrinter { 
    private int rowsPerPage;
    private int columnsPerPage; 
    private int numbersPerPage; 
    private String pageHeader; 
    private PrintStream printStream;

    public RowColumnPagePrinter(int rowsPerPage, int columnsPerPage, String pageHeader) { 
        this.rowsPerPage = rowsPerPage;
        this.columnsPerPage = columnsPerPage; 
        this.pageHeader = pageHeader;
        numbersPerPage = rowsPerPage * columnsPerPage; 
        printStream = System.out;
    }

    public void print(int data[]) { 
        int pageNumber = 1;
        for (int firstIndexOnPage = 0 ; 
            firstIndexOnPage < data.length ; 
            firstIndexOnPage += numbersPerPage) { 
            int lastIndexOnPage =  Math.min(firstIndexOnPage + numbersPerPage - 1, data.length - 1);
            printPageHeader(pageHeader, pageNumber); 
            printPage(firstIndexOnPage, lastIndexOnPage, data); 
            printStream.println("\f");
            pageNumber++;
        } 
    }

    private void printPage(int firstIndexOnPage, int lastIndexOnPage, int[] data) { 
        int firstIndexOfLastRowOnPage =
        firstIndexOnPage + rowsPerPage - 1;
        for (int firstIndexInRow = firstIndexOnPage ; 
            firstIndexInRow <= firstIndexOfLastRowOnPage ;
            firstIndexInRow++) { 
            printRow(firstIndexInRow, lastIndexOnPage, data); 
            printStream.println("");
        } 
    }

    private void printRow(int firstIndexInRow, int lastIndexOnPage, int[] data) {
        for (int column = 0; column < columnsPerPage; column++) {
            int index = firstIndexInRow + column * rowsPerPage; 
            if (index <= lastIndexOnPage)
                printStream.format("%10d", data[index]); 
        }
    }

    private void printPageHeader(String pageHeader, int pageNumber) {
        printStream.println(pageHeader + " --- Page " + pageNumber);
        printStream.println(""); 
    }

    public void setOutput(PrintStream printStream) { 
        this.printStream = printStream;
    } 
}
```

```java
package literatePrimes;

import java.util.ArrayList;

public class PrimeGenerator {
    private static int[] primes;
    private static ArrayList<Integer> multiplesOfPrimeFactors;

    protected static int[] generate(int n) {
        primes = new int[n];
        multiplesOfPrimeFactors = new ArrayList<Integer>(); 
        set2AsFirstPrime(); 
        checkOddNumbersForSubsequentPrimes();
        return primes; 
    }

    private static void set2AsFirstPrime() { 
        primes[0] = 2; 
        multiplesOfPrimeFactors.add(2);
    }

    private static void checkOddNumbersForSubsequentPrimes() { 
        int primeIndex = 1;
        for (int candidate = 3 ; primeIndex < primes.length ; candidate += 2) { 
            if (isPrime(candidate))
                primes[primeIndex++] = candidate; 
        }
    }

    private static boolean isPrime(int candidate) {
        if (isLeastRelevantMultipleOfNextLargerPrimeFactor(candidate)) {
            multiplesOfPrimeFactors.add(candidate);
            return false; 
        }
        return isNotMultipleOfAnyPreviousPrimeFactor(candidate); 
    }

    private static boolean isLeastRelevantMultipleOfNextLargerPrimeFactor(int candidate) {
        int nextLargerPrimeFactor = primes[multiplesOfPrimeFactors.size()];
        int leastRelevantMultiple = nextLargerPrimeFactor * nextLargerPrimeFactor; 
        return candidate == leastRelevantMultiple;
    }

    private static boolean isNotMultipleOfAnyPreviousPrimeFactor(int candidate) {
        for (int n = 1; n < multiplesOfPrimeFactors.size(); n++) {
            if (isMultipleOfNthPrimeFactor(candidate, n)) 
                return false;
        }
        return true; 
    }

    private static boolean isMultipleOfNthPrimeFactor(int candidate, int n) {
        return candidate == smallestOddNthMultipleNotLessThanCandidate(candidate, n);
    }

    private static int smallestOddNthMultipleNotLessThanCandidate(int candidate, int n) {
        int multiple = multiplesOfPrimeFactors.get(n); 
        while (multiple < candidate)
            multiple += 2 * primes[n]; 
        multiplesOfPrimeFactors.set(n, multiple); 
        return multiple;
    } 
}
```

기존 코드보다 프로그램이 길어졌다. 길이가 늘어난 이유는 세 가지다.

1. 리팩터링한 프로그램은 좀 더 길고 서술적인 변수 이름을 사용한다.
2. 리팩터링한 프로그램은 코드에 주석을 추가하는 수단으로 함수 선언과 클래스 선언을 활용한다.
3. 가독성을 높이고자 공백을 추가하고 형식을 맞췄다.

원래 프로그램은 세 가지 책임으로 나눠졌다.

- PrimePrinter

main 함수를 포함하여 실행 환경을 책임진다. 호출 방식이 달라지면 클래스도 바뀐다. 예를들어, 프로그램을 SOAP 서비스로 바꾸려면 PrintPrime 클래스를 고쳐준다.

- RowColumnPagePrinter

숫자 목록을 주어진 행과 열에 맞춰 페이지에 출력하는 방법을 안다. 출력하는 모양세를 바꾸려면 이 클래스를 수정한다.

- PrintGenerator

소수 목록을 생성한다. 소수를 계산하는 알고리즘이 바뀐다면 PrimeGenerator 클래스를 고쳐준다.

재구현이 아니다. 프로그램을 처음부터 다시 짜지 않았다.

- 원래 프로그램의 정확한 동작을 검증하는 테스트 슈트를 작성한다.
- 한번에 하나씩 수 차례에 걸쳐 조금씩 코드를 변경한다.
- 코드를 변경할 때마다 테스트를 수행해 원래 프로그램과 동일하게 동작하는지 확인한다.

## 변경하기 쉬운 클래스

시스템은 변경이 불가피하다. 그리고 변경이 있을 때 마다 의도대로 동작하지 않을 위험이 따른다.

깨끗한 시스템은 클래스를 체계적으로 관리해 변경에 따르는 위험을 최대한 낮춘다.

예를 들어 주어진 메타 자료로 적절한 SQL을 만드는 클래스가 있다고 해보자. 이 클래스는 아직 미완성이라 update 문과 같은 일부의 SQL을 지원하지 않는다. 언젠가 update를 지원해야 한다면 클래스를 손대어 고쳐야만 한다.

문제는 코드를 손대면 위험이 생긴다. 어떤 변경이든 클래스를 손대면 다른 코드를 망가뜨릴 잠정적인 위험이 존재한다. 그래서 테스트도 완전히 다시 해야만 한다

```java
public class Sql {
    public Sql(String table, Column[] columns)
    public String create()
    public String insert(Object[] fields)
    public String selectAll()
    public String findByKey(String keyColumn, String keyValue)
    public String select(Column column, String pattern)
    public String select(Criteria criteria)
    public String preparedInsert()
    private String columnList(Column[] columns)
    private String valuesList(Object[] fields, final Column[] columns) private String selectWithCriteria(String criteria)
    private String placeholderList(Column[] columns)
}
```

이를 수정하지 않으려면 어떻게 해야 할까?

```java
abstract public class Sql {
    public Sql(String table, Column[] columns) 
    abstract public String generate();
}
public class CreateSql extends Sql {
    public CreateSql(String table, Column[] columns) 
    @Override public String generate()
}

public class SelectSql extends Sql {
    public SelectSql(String table, Column[] columns) 
    @Override public String generate()
}

public class InsertSql extends Sql {
    public InsertSql(String table, Column[] columns, Object[] fields) 
    @Override public String generate()
    private String valuesList(Object[] fields, final Column[] columns)
}

public class SelectWithCriteriaSql extends Sql { 
    public SelectWithCriteriaSql(
    String table, Column[] columns, Criteria criteria) 
    @Override public String generate()
}

public class SelectWithMatchSql extends Sql { 
    public SelectWithMatchSql(String table, Column[] columns, Column column, String pattern) 
    @Override public String generate()
}

public class FindByKeySql extends Sql public FindByKeySql(
    String table, Column[] columns, String keyColumn, String keyValue) 
    @Override public String generate()
}

public class PreparedInsertSql extends Sql {
    public PreparedInsertSql(String table, Column[] columns) 
    @Override public String generate() {
    private String placeholderList(Column[] columns)
}

public class Where {
    public Where(String criteria) public String generate()
}

public class ColumnList {
    public ColumnList(Column[] columns) public String generate()
}
```

이렇게 수정한다면 update문을 추가할 때 기존 클래스를 변경할 필요가 전혀 없다.

클래스가 서로 분리되어있기 때문이다. 단지 update문을 만들기 위해서는 Sql 클래스에서 새 클래스 UpdateSql을 상속받아 거기에 넣으면 그만이다.

이렇게 짜면 다른 코드가 망가질 염려도 없다. 또한 OCP 원칙 또한 준수하고 있다

<aside>
💡 OCP(Open-Closed-Principle)  

확장에 개방적이고 수정에 폐쇄적이어야 한다.
</aside>

### 변경으로부터 격리

요구사항은 변하기 마련이다. 따라서 코드도 변하기 마련이다.

- 구체 클래스
  - 상세한 구현 코드를 포함한다.
- 추상 클래스
  - 개념만 포함한다.

상세한 구현에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험에 빠진다. 그래서 우리는 인터페이스와 추상 클래스를 사용해 구현이 미치는 영향을 격리한다.

상세한 구현에 의존하는 코드는 테스트가 어렵다.

예를 들어 아래의 Portfolio 클래스는 외부 TokyoStockExchange API를 사용해 포트폴리오 값을 계산한다. 따라서 테스트코드는 시세 변화에 영향을 받는다. 5분마다 값이 달라지는 API로 테스트 코드를 짜기란 쉽지 않다.

```java
public interface StockExchange { 
    Money currentPrice(String symbol);
}
```

다음으로 StockExchange 인터페이스를 구현하는 TokyoStockExchange 클래스를 구현한다. 또한 Portfolio 생성자를 수정해 StockExchange 참조자를 인수로 받는다.

```java
public Portfolio {
    private StockExchange exchange;
    public Portfolio(StockExchange exchange) {
        this.exchange = exchange; 
    }
    // ... 
}
```

이제 TokyoStockExchange 클래스를 흉내내는 테스트용 클래스를 만들 수 있다.

테스트용 클래스는 StockExchange 인터페이스를 구현하며 고정된 주가를 반환한다.

테스트에서 마이크로소프트 주식 다섯 주를 구입하면 테스트용 클래스는 주가로 언제나 100불을 반환한다. 따라서 우리는 전체 포트폴리오 총계가 500불인지 확인하는 테스트 코드를 작성할 수 있다.

```java
public class PortfolioTest {
    private FixedStockExchangeStub exchange;
    private Portfolio portfolio;

    @Before
    protected void setUp() throws Exception {
        exchange = new FixedStockExchangeStub(); 
        exchange.fix("MSFT", 100);
        portfolio = new Portfolio(exchange);
    }

    @Test
    public void GivenFiveMSFTTotalShouldBe500() throws Exception {
        portfolio.add(5, "MSFT");
        Assert.assertEquals(500, portfolio.value()); 
    }
}
```

이렇게 테스트가 가능할 정도로 시스템의 결합도를 낮추면 유연성과 재사용성도 더욱 높아진다.

결합도가 낮아진다는 소리는 각 시스템 요소가 다른 요소로부터 그리고 변경으로부터 잘 격리되어 있다는 의미다. 시스템 요소가 서로 잘 격리되어 있으면 각 요소를 이해하기도 쉬워진다.

결합도를 최소로 줄이면 자연스럽게 또 다른 클래스 설계 원칙인 DIP(Dependency Inversion Principle)를 따르는 클래스가 나온다. DIP는 클래스가 상세한 구현이 아니라 추상화에 의존해야 한다는 원칙이다.

## 참조

> 로버트 C. 마틴, 『Clean Code 클린 코드』, 박재호, 이해영 옮김, 인사이트(2021), P172-190.
>