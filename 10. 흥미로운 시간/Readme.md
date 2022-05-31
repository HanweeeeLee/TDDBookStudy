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
그래도 에러가 발생한다 "excepted"
