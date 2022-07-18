# 30. 디자인 패턴
이 책에서 다루는 패턴에 대한 요약

 - 커맨드: 계산 작업데 대한 호출을 메시지가 아닌 객체로 표현한다.
 - 값 객체: 객체가 생성된 이후 그 값이 절대로 변하지 않게 하여 별칭 문제가 발생하지 않게 한다.
 - 널 객체: 계산 작업의 기본 사례를 객체로 표현한다.
 - 탬플릿 메서드: 계산 작업의 변하지 않는 순서를 여러 추상 메서드로 표현한다. 이 추상 메서드들은 상속을 통해 특별한 작업을 수행하게끔 구체화 된다.
 - 플러거블 객체: 둘 이상의 구현을 객체를 호출함으로써 다양성을 표현한다.
 - 팩토리 메서드: 생성자 대신 메서드를 호출함으로써 객체를 생성한다.
 - 임포스터: 현존하는 프로토콜을 갖는 다른 구현을 추가하여 시스템에 변이를 도입한다.
 - 컴포지드: 하나의 객체로 여러 객체의 행위 조합을 표현한다.
 - 수집 매개 변수: 여러 다른 객체에서 계산한 결과를 모으기 위해 매개 변수를 여러 곳으로 전달한다.

![KakaoTalk_Photo_2022-07-18-23-12-30](https://user-images.githubusercontent.com/60125719/179530757-53c0b9e9-b66c-4dc7-9618-fb16232a1a9e.jpeg)

## 커맨드
간단한 메서드 호출보다 복잡한 형태의 계산 작업데 대한 호출이 필요하다면? 계산 작업에 대한 객체를 생성하여 이를 호출하자.  
복잡한 계산 작업 호출은 값비싼 메커니즘을 필요로 한다. 호출 자체를 나타내기 위한 객체를 만들고, run()과 같은 프로토콜을 이용해 계산을 호출하자. 
```Java
// Runnable
interface Runnable
    public abstract void run();
```
> run()의 구현으로는 어떤것이든 넣으면 된다. 

## 값 객체
널리 공유해야 하지만 동일성(identity)은 중요하지 않을 때 객체를 어떤 식으로 설계할 수 있을까? 객체가 생성될 때 객체의 상태를 설정한 후 이 상태가 절대 변할 수 없도록 하자. 그리고 이 객체에 대해 수행되는 연산은 언제나 새로운 객체를 반환하게 만든다.  
```
'나'라는 어떤 객체가 있는데, '내'가 Rectangle 객체를 갖는다고 가정해보자. 나는 Rectangle에 기반을 두고 면적 등을 계산하는데, 누군가가 Rectangle 객체를 빌려달라고 했다.
그래서 빌려주고, 돌려받았는데, 내가 모르는 사이 Rectangle객체의 상태값이 변해있다. 그래서 내가 계산한 면적값이 이상해졌는데 내가 그 사실을 알 수 있는 방법은 없다.
```
> 고전적인 별칭 문제. 두 객체가 제삼의 다른 객체에 대한 참조를 공유하고있는데, 한 객체가 공유하는 객체의 상태를 바꾸면 다른 객체는 그 사실을 알 수 없다.   
  
해결법:  
1. 현재 의존하는 객체에 대한 참조를 결코 외부로 알리지 않는다. 그 대신 복사본을 제공한다.  
2. 옵저버 패턴을 이용한다. 
3. 객체를 덜 객체답게 취급한다. (변하지 않도록. 상수처럼, '값'으로 만든다.)  
3-1. 모든 값 객체는 동등성을 구현해야 한다. (5달러가 5달러와 같듯이)

## 널 객체
객체의 특별한 상황을 표현하고자 할때는 그 특별한 상황을 표현하는 새로운 객체를 만들자.  

```Java
class File {
    boolean setReadOnly() {
      SecurityManager guard = System.getSecurityManager();
      if (guard != null) {
         guard.canWrite(path);
      }
      return fileSystem.setReadOnly(this);
    }    
}

```
-> 절대 예외를 던지지 않는 새로운 클래스를 만들자
```JAVA
class LaxSecurity {
    public void canWrite(String path) {

    }
    public static SecurityManager getSecurityManager() {
        return security == null ? new LaxSecurity() : security;
    }
}

class File {
    boolean setReadOnly() {
        // SecurityManager guard = System.getSecurityManager();
        // if (guard != null) {
        //     guard.canWrite(path);
        // }
        SecurityManager security = System.getSecurityManager();
        security.canWrite(path);
        return fileSystem.setReadOnly(this);
    }    
}
```

## 탬플릿 메서드
작업 순서는 변하지 않지만 각 작업 단위에 대한 미래의 개선 가능성을 열어두고 싶은 경우엔? 다른 메서드들을 호출하는 내용으로만 이루어진 메서드를 만들자.  
프로그래밍이란 고전적인 순서들로 가득하다.
 - 입력/처리/출력
 - 메시지 보내기/응답 받기
 - 명령 읽기/결과 내보내기
  
이러한 순서들의 범용성에 대해서 명백하게 나타내는 동시에 각 단계의 구현에 대해서는 변화를 주고싶은 경우가 있다.

```JAVA
class TestCase {
    public void runBare() throws Throwable {
        setUp();
        try {
            runTest();
        }
        finally {
            tearDown();
        }
    }
}
```
> 하위 클래스는 그들이 원하는 대로 setUp(), runTest(), tearDown()을 구현하면 된다.  
  
탬플릿 메서드를 만들 때 한 가지 문제는 하위 클래스를 위한 기본 구현을 제공할 것인가 말 것인가 하는 것이다.  

## 플러거블 객체
변이를 어떻게 표현할 것인가? 가장 간단한 방법은 명시적인 조건문을 사용하는 것이다.  
```Java
if (circle) then {
    ... circley stuff ...
} else {
    ... non circly stuff
}
```
원가 원이 아닌 것을 구분하기 위해 한 곳에서 명시적인 조건문을 사용하게 되면, 조건문은 퍼져나간다.  
TDD의 두 번쨰 수칙이 중복을 제거하는 것이기 때문에, 명시적인 조건문이 전염되는 싹을 잘라야한다.  


ex) 마우스 버튼을 누를 때 어떤 도형 위에 있는 경우, 마우스를 움직이면 도형이 이동. 마우스를 놓으면 도형이 선택.  
도형 위에 있지 않은 경우, 마우스 버튼을 누르면 도형 집합을 선택. 그 이후 마우스를 움직이면 여러 도형을 선택하는 데 사용된 사각형의 크기를 변경
```JAVA
class selectionTool {
    Figure selected;
    public void mouseDown() {
        selected = findFigure();

        if (selected != null) 
            select(selected)
    }

    public void mouseMove() {
        if (selected != null)
            move(selected);
        else 
            moveSelectionRectangle();
    }

    public void mouseUp() {
        if (selected == null)
            selectedAll();
    }
}
```
-> 
```Java
class selectionTool {
    SelectedMode = mode;
    public void mouseDown() {
        selected = findFigure();

        if (selected != null) 
            mode = SingleSelection(selected);
        else 
            mode = MultipleSelection();
    }

    public void mouseMove() {
        mode.mouseMove();
    }

    public void mouseUp() {
        mode.mouseUp();
    }
}
```

## 플러거블 셀렉터
인스턴스별로 서로 다른 메서드가 동적으로 호출되게 하려면? 메서드의 이름을 저장하고 있다가 그 이름에 해당하는 메서드를 동적으로 호출하자.  

```Java
abstract class Report {
    abstract void print();
}

class HTMLReport extends Report {
    void print() {
        ...
    }
}

class XMLReport extends Report {
    void print() {
        ...
    }
}
```
->
```Java
abstract class Report {
    String printMessage;
    Report(String printMessage) {
        this printMessage = printMessage;
    }
    void print() {
        switch (printMessage) {
        case "printHTML":
            printHTML();
            break;
        case "printXML":
            printXML();
            break;
        }
    }

    void printHTML() {

    }

    void printXML() {

    }
}
```
> 새로운 종류의 출력을 추가할 떄마다 출력 메서드를 추가하고 switch문을 바꿔야 한다는 점을 기억하라.

->
```JAVA
void print() {
    Method runMethod = getClass().getMethod(printMessage, null);
    runMethod.invoke(this, new Class[0]);
}
```
> 최소한 switch문은 없다.  

플러거블 셀렉터의 문제는 메서드가 호출되었는지 보기 위해 코드를 추적하는것.  
확실히 직관적인 상황에서 코드를 정리하기 위한 용도로만 사용하자. 

## 팩토리 메서드
새 객체를 만들 때 유연성을 원하는 경우 객체를 어떻게 생성? 생성자를 쓰는 대신 일반 메서드에서 객체를 생성하자.  
  
```Java
public void testMultiplication() {
    Dollar five = new Dollar(5);
    assertEquals(new Dollar(10), five.times(2));
    assertEquals(new Dollar(15), five.times(3));
}
```
-> 
```Java
public void testMultiplication() {
    // Dollar five = new Dollar(5);
    Dollar five = Money.dollar(5);
    assertEquals(new Dollar(10), five.times(2));
    assertEquals(new Dollar(15), five.times(3));
}

class Money {
    static Dollar dollar(int amount) {
        return new Dollar(amount);
    }
}
```
> 단점은 인디렉션. 메서드가 생성자처럼 생기지는 않았지만 객체를 생성한다는 사실을 기억하고 있어야한다. 

## 사칭 사기꾼
기존 코드에 새로운 변이를 도입하려면 어떻게 해야 할까? 기존의 객체와 같은 프로토콜을 갖지만 구현은 다른 새로운 객체를 추가한다.  
  
ex) 우리가 그래픽 에디터를 테스트하고 있고, 이미 사각형을 제대로 그린다고 가정하자. 
```Java
testRectangle() {
    Drawing d = new Drawing();
    d.addFigure(new RactangleFigure(0, 10, 50, 100));
    RecodingMedium brush = new RecodingMedium();
    d.display(brush);
    assertEquals("rectangle 0 10 50 100\n", brush.log());
}
```

이제 타원을 표시하고 싶다. 이 경우, 사칭 사기꾼(imposter)을 발견하기는 쉽다. RectangleFigure를 OvalFigure로 바꾸면 된다.
```Java
testOval() {
    Drawing d = new Drawing();
    d.addFigure(new OvalFigure로(0, 10, 50, 100));
    RecodingMedium brush = new RecodingMedium();
    d.display(brush);
    assertEquals("rectangle 0 10 50 100\n", brush.log());
}
```
다음은 리팩토링 중에 나타나는 사칭 사기꾼의 두 가지 예다. 
 - 널 객체: 데이터가 없는 상태를 데이터가 있는 상태와 동일하게 취급할 수 있다. 
 - 컴포지드: 객체의 집합을 단일 객체처럼 취급할 수 있다. 


## 컴포지드 
하나의 객체가 다른 객체 목록의 행위를 조합한 것처럼 행동하게 만들려면 어떻게 해야 할까? 객체 집합을 나타내는 객체를 단일 객체에 대한 임포스터로 구현한다.  
```Java
class Transaction {
    Transaction(Money value) {
        this.value = value;
    }
}

class Account {
    Transaction transactions[];
    Money balance() {
        Money sum = Money.zero();
        for (int i = 0 ; i < transactions.length ; i++) 
            sum = sum.plus(transactions[i].value);
        return sum;
    }
}
```
> 1. Transaction은 값을 갖는다. 2. Account는 잔액을 갖는다.  
  
고객은 여러 계좌를 가지고 있고 전체 계좌의 잔액을 알고싶어한다. 이를 구현하는 방법 한 가지는 새로운 클래스인 OverallAccount를 만드는것. OverallAccount는 모든 Account의 잔액을 합친다. -> 중복  
Account와 Balance가 동일한 인터페이스를 갖게 만들면?  

```Java
interfcae Holding
    Money balance();
```
Transaction에서는 balance()가 value를 반환하게 만들면 된다.
```JAVA
class Transaction {
    Money balance() {
        return value;
    }
}
```
이제 Account는 Transaction이 아닌 Holding의 조합으로 만들 수 있다. 
```Java
class Account {
    Holding holdings[];
    Money balance() {
        Money sum = Money.zero();
        for (int i = 0 ; i < holdings.length ; i++) 
            sum = sum.plus(holdings[i].balance());
        return su,;
    }
}
```
> 이제 OverallAccount에 대한 문제는 사라졌다. OverallAccount는 단지 Account를 담고 있는 Account일 뿐이다.

## 수집 매개 변수
여러 객체에 걸쳐 존재하는 오퍼레이션의 결과를 수집하려면 어떻게 해야할까?




































