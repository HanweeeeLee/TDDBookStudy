# 09. 우리가 사는 시간

## TODO)
 - $5 + 10CHF = $10(환율이 2:1일 경우)
 - ~~$5 X 2 = $10~~
 - ~~amount를 private으로 만들기~~
 - ~~Dollar 부작용?~~
 - Money 반올림?
 - ~~equals()~~
 - hasCode()
 - Equal null
 - Equal Object
 - ~~5CHF X 2 = 10CHF~~
 - **Dollar/Franc 중복**
 - ~~공용 equals~~
 - 공용 times
 - ~~Franc과 Dollar 비교하기~~
 - **통화?**
 - testFrancMultiplication을 지워야 할까?

불필요하고 귀찮은 하위 클래스를 제거하기 위해서 통화 개념을 도입해보자.  
우선 Money 클래스에 currency() 메서드를 선언해보자.
```JAVA
abstract class Money {
    protected int amount;
    abstract Money times(int multiplier);
    abstract String currency();
    
    public boolean equals(Object object) {
        Money money = (Money) object;
        return amount == money.amount && getClass().equals(money.getClass());
    }

    static Money dollar(int amount) {
        return new Dollar(amount);
    }

    static Money franc(int amount) {
        return new Franc(amount);
    }
}
```
그리고 두 하위클래스인 Dollar와 Franc에서 이를 구현해보자
```JAVA
// Dollar 클래스
...
String currency() {
  return "CHF";
}
...

// Franc 클래스
...
String currency() {
  return "USD";
}
...
```
우리는 두 클래스를 모두 포함할 수 있는 동일한 구현을 원한다.  
통화를 인스턴스 변수에 저장하고, 메서드에서는 그것을 반환하게 만들어 보자.
```JAVA
// Dollar 클래스
private String currency;

Dollar(int amount) {
  this.amount = amount;
  currency = "USD";
}

String currency() {
  return currency;
}

// Franc 클래스
private String currency;

Franc(int amount) {
  this.amount = amount;
  currency = "CHF";
}

String currency() {
  return currency;
}
```
이제 두 currency()가 동일하므로 변수 선언과 currency()구현을 둘 다 위로 올릴수 있게 됐다.
```JAVA
// Money 클래스
protected String currency;

String currency() {
  return currency;
}
```
문자열 USD 와 CHF 를 정적 팩토리 메서드로 옮긴다면 두 생성자가 동일해질 것이고, 공통구현을 만들수 있을것이다.  
우선 생성자에 인자를 추가하자
```JAVA
// Franc 클래스
Franc(int amount, String currency) {
  this.amount = amount;
  this.currency = "CHF";
}
```
이렇게 변경하게 되면 생성자를 호출하게 되는 두곳이 깨진다.

```JAVA
// Money 클래스
static Money franc(int amount) {
  return new Franc(amount, null);   //생성자를 호출하는 부분
}

// Franc 클래스
Money times(int multiplier) {
  return new Franc(amount * multiplier, null);  //생성자를 호출하는 부분
}
```
일단 먼저 times()를 정리하도록 하자.
```JAVA
// Franc 클래스
Money times(int multiplier) {
  return Money.franc(amount * multiplier);
}
```
그 후 이제 팩토리 메서드가 CHF를 전달 할 수 있게된다.
```JAVA
// Money 클래스
static Money franc(int amount) {
  return new Franc(amount, "CHF");
}
```
마지막으로 인자를 인스턴스 변수에 할당할수 있게 된다.
```JAVA
// Franc 클래스
Franc(int amount, String currency) {
  this.amount = amount;
  this.currency = currency;
}
```
이제 Franc을 진행했던것과 마찬가지로 Dollar를 수정해준다.  
전체적인 코드는 아래와 같다.
```JAVA
// Money 클래스
abstract class Money {
    protected int amount;
    abstract Money times(int multiplier);
    protected String currency;

    String currency() {
        return currency;
    }

    public boolean equals(Object object) {
        Money money = (Money) object;
        return amount == money.amount && getClass().equals(money.getClass());
    }

    static Money dollar(int amount) {
        return new Dollar(amount, "USD");
    }

    static Money franc(int amount) {
        return new Franc(amount, "CHF");
    }
}

// Franc 클래스
public class Franc extends Money{
    private String currency;
    public Franc(int amount) {
        this.amount = amount;
    }

    Money times(int multiplier) {
        return Money.franc(amount * multiplier);
    }

    Franc(int amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }
    
    @Override
    String currency() {
        return currency;
    }

    public boolean equals(Object object) {
        Money money = (Money) object;
        return amount == money.amount;
    }
}

// Dollar 클래스
public class Dollar extends Money {
    private String currency;
    public Dollar(int amount) {
        this.amount = amount;
    }

    Money times(int multiplier) {
        return Money.dollar( amount * multiplier);
    }

    Dollar(int amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }
    
    @Override
    String currency() {
        return currency;
    }

    public boolean equals(Object object) {
        Money money = (Money) object;
        return amount == money.amount;
    }
}
```
최종적으로 두 생성자가 이제 동일해졌다. 구현을 상위 클래스로 올리도록 하자.
```JAVA
// Money 클래스
abstract class Money {
    protected int amount;
    abstract Money times(int multiplier);
    protected String currency;

    String currency() {
        return currency;
    }

    public boolean equals(Object object) {
        Money money = (Money) object;
        return amount == money.amount && getClass().equals(money.getClass());
    }

    Money(int amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }

    static Money dollar(int amount) {
        return new Dollar(amount, "USD");
    }

    static Money franc(int amount) {
        return new Franc(amount, "CHF");
    }
}

// Franc 클래스
public class Franc extends Money{
    private String currency;

    Money times(int multiplier) {
        return Money.franc(amount * multiplier);
    }
    
    Franc(int amount, String currency) {
        super(amount, currency);
    }
    
    @Override
    String currency() {
        return currency;
    }

    public boolean equals(Object object) {
        Money money = (Money) object;
        return amount == money.amount;
    }
}

// Dollar 클래스
public class Dollar extends Money {
    private String currency;

    Money times(int multiplier) {
        return Money.dollar( amount * multiplier);
    }

    Dollar(int amount, String currency) {
        super(amount, currency);
    }
    
    @Override
    String currency() {
        return currency;
    }

    public boolean equals(Object object) {
        Money money = (Money) object;
        return amount == money.amount;
    }
}

## 결론
 - 다른 부분들(통화)을 호출자로 옮겨서 두 생성자를 일치시켰다.
 - times()가 팩토리 메서드를 사용하도록 만들기 위해 리팩토링을 잠시 중단했다.
 - 동일한 생성자들을 상위 클래스로 옮겼다.

## TODO)
 - $5 + 10CHF = $10(환율이 2:1일 경우)
 - ~~$5 X 2 = $10~~
 - ~~amount를 private으로 만들기~~
 - ~~Dollar 부작용?~~
 - Money 반올림?
 - ~~equals()~~
 - hasCode()
 - Equal null
 - Equal Object
 - ~~5CHF X 2 = 10CHF~~
 - **Dollar/Franc 중복**
 - ~~공용 equals~~
 - 공용 times
 - ~~Franc과 Dollar 비교하기~~
 - ~~**통화?**~~
 - testFrancMultiplication을 지워야 할까?
