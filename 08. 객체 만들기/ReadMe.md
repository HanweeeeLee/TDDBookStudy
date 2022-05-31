# 08. 객체 만들기

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
 - 통화?

현재 Franc와 Dollar의 times() 구현코드가 비숫하다.

```JAVA
//Franc 클래스
public Franc times(int multiplier) {
  return new Franc(amount * multiplier);
}

//Dollar 클래스
public Dollar times(int multiplier) {
  return new Dollar(amount * multiplier);
}
```

양쪽 모두를 Money를 반환하게 만들면 더욱 비슷하게 만들수 있다.
```JAVA
//Franc 클래스
public Money times(int multiplier) {
  return new Franc(amount * multiplier);
}

//Dollar 클래스
public Money times(int multiplier) {
  return new Dollar(amount * multiplier);
}
```
Money의 두 하위 클래스는 그다지 많은 일을 하는거 같지 않으므로 제거하고 싶다.  
한번에 없애 버리고 싶지만 너무 큰 일이라 작은 일부터 테스트로 수행해가는 TDD의 목적에 알맞지 않다.
그러므로 작은 단위로서 하위 클래스에 대한 직접적인 참조를 제거해 보도록 하자.
일단 Money에 Dollar를 반환하는 팩토리 메서드를 도입해보자
```JAVA
@Test
void testMultiplication() {
  Dollar five = Money.dollar(5);
  assertEquals(new Dollar(10), five.times(2));
  assertEquals(new Dollar(15), five.times(3));
}

//Money 클래스
static Dollar dollar(int amount) {
  return new Dollar(amount);
}
```
Dollar에 대한 참조가 사라지기를 바라므로 테스트를 다음과 같이 변경하자
```JAVA
void testMultiplication() {
  Money five = Money.dollar(5);
  assertEquals(new Dollar(10), five.times(2));
  assertEquals(new Dollar(15), five.times(3));
}
```
컴파일을 진행해보면 Money에는 times()가 정의되지 않았다고 알려준다.  
여기서 Money를 추상 클래스로 변경한 후 Money.times()를 선언하자
```JAVA
abstract class Money {
  protected int amount;
  abstract Money times(int multiplier);  

  static Money dollar(int amount) {
    return new Dollar(amount);
  }
  //생략...
}
```
이제 팩토리 메서드의 선언을 바꿀수 있다.
테스트 코드의 나머지 모든곳들에서 사용하도록 변경하자. 기존의 Dollar나 Franc을 전부 Money로 변경해준다.
```JAVA
@Test
public void testMultiplication() {
  Money five = Money.dollar(5);
  assertEquals(Money.dollar(10), five.times(2));
  assertEquals(Money.dollar(15), five.times(3));
}

@Test
public void testEquality() {
  assertTrue(Money.Dollar(5).equals(Money.Dollar(5)));
	assertFalse(Money.Dollar(5).equals(Money.Dollar(6)));
	assertTrue(Money.Franc(5).equals(Money.Franc(5)));
	assertFalse(Money.Franc(5).equals(Money.Franc(6)));
}

```
이제 어떤 클라이언트 코드도 Dollar라는 이름의 하위 클래스가 있다는걸 알지 못한다.  
하위 클래스의 존재를 테스트에서 분리함으로서 어떤 모델 코드에도 영향을 주지 않고 상속 구조를 마음껏 변경할수 있게 되었다.  
그런데 testFrancMultiplication을 변경하려다 보니 이 테스트가 검사하는 로직 중 Dollar 곱하기 테스트에 의해  
검사되지 않는 부분은 하나도 없다는것을 눈치챘다.  
이 테스트를 삭제하면 코드에 대한 확신이 조금이라도 줄어 들수도 있기에 일단은 그대로 두자. 하지만 조금 수상쩍다.
```JAVA
@Test
public void testMultiplication() {
  Money five = Money.dollar(5);
  assertEquals(Money.dollar(10), five.times(2));
  assertEquals(Money.dollar(15), five.times(3));
}

@Test
public void testEquality() {
  assertTrue(Money.Dollar(5).equals(Money.Dollar(5)));
	assertFalse(Money.Dollar(5).equals(Money.Dollar(6)));
	assertTrue(Money.Franc(5).equals(Money.Franc(5)));
	assertFalse(Money.Franc(5).equals(Money.Franc(6)));
}

@Test
public void testFrancMultiplication() {
  Money five = Money.Franc(5);
  assertEquals(Money.Franc(10), five.times(2));
  assertEquals(Money.Franc(15), five.times(3));
}

//Money 클래스
```JAVA
abstract class Money {
  protected int amount;
  abstract Money times(int multiplier);  

  static Money dollar(int amount) {
    return new Dollar(amount);
  }
  
  static Money Franc(int amount) {
    return new Franc(amount);
  }
  //생략...
}
```
## 결론
다음장에서 우리는 times()의 중복을 거둬낼것이다. 하지만 일단은 검토해보자면  
 - 동일한 메서드(times)의 두 변이형 메서드 서명부를 통일시킴으로서 중복 제거를 향해 한단계 더 전진했다.
 - 최소한 메서드 선언부만이라도 공통상위클래스(superclass)로 옮겼다.
 - 팩토리 메서드를 도입하여 테스트 코드에서 콘크리트 하위 클래스의 존재 사실을 분리해냈다.
 - 하위 클래스가 사라지면 몇몇 테스트는 불필요한 여분의 것이 된다는 것을 인식했다. 일단은 그냥 두었다.

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
 - 통화?
 - testFrancMultiplication을 지워야 할까?
