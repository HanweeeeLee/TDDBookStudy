# 10. 흥미로운 시간

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
 - Dollar/Franc 중복
 - ~~공용 equals~~
 - **공용 times**
 - ~~Franc과 Dollar 비교하기~~
 - ~~통화?~~
 - testFrancMultiplication을 지워야 할까?

이 장에서는 Money를 나타내기 위한 단 하나의 클래스만을 가지게 될 것이다.  
두 times()의 구현이 거의 비슷하기는 하지만 아직 완전히 동일하지는 않다.
```JAVA
// Franc
public class Franc extends Money{
  //...
  Money times(int multiplier) {
    return Money.Franc(amount * multiplier);
  }
  //...
}

// Dollar
public class Dollar extends Money {
  //...
  Money times(int multiplier) {
    return Money.Dollar(amount * multiplier);
  }
  //...
}
```
이 둘을 동일하게 만들 명백한 방법이 떠오르지가 않는다.  
때로는 전진하기 위해 물러서야 할 때도 있다. 일단 팩토리 메서드를 다시 인라인해보자.
```JAVA
// Franc 클래스
public class Franc extends Money{
  //...
  Money times(int multiplier) {
    return new Franc(amount * multiplier, "CHF");
  }
  //...
}

// Dollar 클래스
public class Dollar extends Money {
  //...
  Money times(int multiplier) {
    return new Dollar( amount * multiplier, "USD");
  }
  //...
}
```
Franc에서는 변수 currency가 항상 'CHF' 이고 Dollar에서는 항상 'USD'이기에 다음과 같이 변경한다.
```JAVA
// Franc 클래스
public class Franc extends Money{
  //...
  Money times(int multiplier) {
    return new Franc(amount * multiplier, currency);
  }
  //...
}

// Dollar 클래스
public class Dollar extends Money {
  //...
  Money times(int multiplier) {
    return new Dollar( amount * multiplier, currency);
  }
  //...
}
```
코드를 바꾸고 생각해보니 Franc을 반환할지 Money를 반환할지가 중요한 문제인가?  
때로는 고민보다 코드를 작성해서 컴퓨터에게 물어보는 것이 빠르다.  
Franc.times()가 Money를 반환하도록 고쳐보자.
```JAVA
// Franc 클래스
public class Franc extends Money{
  //...
  Money times(int multiplier) {
    return new Money(amount * multiplier, currency);
  }
  //...
}
```
컴파일시 컴파일러가 Money클래스는 콘크리트 클래스여야 한다고 한다. 고쳐주자
```JAVA
// Money 클래스
class Money {
  //...
  Money times(int multiplier) {
    return null;
  }
  //...
}
```
그래도 에러가 발생한다 "excepted:<Money.Franc@31aebf> but was <Money.Money@478a43>"
무슨 메세지인지 확인해보기 위해 toString()을 정의하자.
```JAVA
// Money 클래스
class Money {
  //...
  Public String to String(){
   return amount + " " + currency;
  }
  //...
}
```
toString()을 작성하기전에 테스트를 작성하는게 맞지만  
 - 화면에 나타나는 결과를 보려던 참이고
 - toString()은 디버그 출력에만 쓰이기에 잘못 구현됨으로 얻을 리스크가 적고
 - 이미 에러 상태에서는 새로운 테스트를 추가하지 않는 편이 좋을거 같다.

이제 에러메시지에 "excepted:<10 CHF> but was <10 CHF>"라고 나오게된다.  
답은 맞지만 클래스가 다르다. 문제는 equals() 구현에 있다.
```JAVA
// Money 클래스
class Money {
  //...
  public boolean equals(Object object) {
   Money money = (Money) object;
   return amount == money.amount && getClass().equals(money.getClass());
  }
  //...
}
```
위 코드를 보면 우리가 정말로 검사해야 할 것은 클래스가 같은지가 아니라 currency가 같은지의 여부이다.  
위에서 말했듯이 빨간 막대 상태일때에는 새로운 테스트를 추가하지 않는것이 좋다.  
이번에는 보수적인 방법인 변경된 코드를 되돌려서 다시 초록막대 상태로 돌아가는 방법을 택하자.
```JAVA
// Franc 클래스
public class Franc extends Money{
  //...
  Money times(int multiplier) {
    return new Franc(amount * multiplier, currency);
  }
  //...
}
```
일단 다시 초록막대 상태로 돌아왔다.  
우리는 Franc(10, "CHF")와 Money(10, "CHF")가 서로 같기를 원하지만 그렇지 않다고 보고된 것이다.  
그래서 이 부분을 그대로 테스트로 사용할 수 있다.
```JAVA
public void testDifferentClassEquality(){
 assertTrue(new Money(10, "CHF").equals(new Franc(10, "CHF")));
}
```
예상대로 실패하게 된다.  
equals() 코드는 클래스가 아니라 currency를 비교해야 한다.
```JAVA
// Money 클래스
class Money {
  //...
  public boolean equals(Object object) {
   Money money = (Money) object;
   return amount == money.amount && currency().equals(money.currency());
  }
  //...
}
```
이제 Franc.times() 에서 Money를 반환해도 테스트가 통과하게 할 수 있다.
```JAVA
// Franc 클래스
public class Franc extends Money{
  //...
  Money times(int multiplier) {
    return new Money(amount * multiplier, currency);
  }
  //...
}
```
이제 Dollar.times()에서도 통과되는지 확인한다.
```JAVA
// Dollar 클래스
public class Dollar extends Money{
  //...
  Money times(int multiplier) {
    return new Money(amount * multiplier, currency);
  }
  //...
}
```
무사히 통과되고, 이제 두 구현이 동일해 졌으니 상위 클래스로 끌어올릴수 있다.
```JAVA
// Money 클래스
class Money {
  //...
  Money times(int multiplier){
   return new Money(amount * multiplier, currency);
  }
  //...
}
```
이제, 아무것도 안하는 멍청한 하위클래스들을 제거할 수 있게 된다.

## 결론
 - 두 times()를 일치시키기 위해 그 메서드들이 호출하는 다른 메서드들을 인라인 시킨 후 상수를 변수로 바꿔 주었다.
 - 단지 디버깅을 위해 toString()을 작성했다.
 - Franc 대신 Money를 반환하는 변경을 시도한 뒤 그것이 잘 작동할지를 테스트가 말하도록 했다.
 - 실험해본 걸 뒤로 물리고 또 다른 테스트를 작성했다. 테스트를 작동했더니 실험도 제대로 작동했다.

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
 - Dollar/Franc 중복
 - ~~공용 equals~~
 - ~~**공용 times**~~
 - ~~Franc과 Dollar 비교하기~~
 - ~~통화?~~
 - testFrancMultiplication을 지워야 할까?
